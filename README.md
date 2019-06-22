```bash

timedatectl set-ntp true

fdisk -l

fdisk /dev/sda
	g
	n 1 +1G
	t L 1
	n 2 +350G
	p
	w

fdisk -l

# Encryption https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_an_entire_system#LUKS_on_a_partition
cryptsetup -y -v luksFormat /dev/sda2
cryptsetup open /dev/sda2 cryptroot
mkfs.ext4 /dev/mapper/cryptroot
mount /dev/mapper/cryptroot /mnt
# Check the mapping works as intended:
umount /mnt
cryptsetup close cryptroot
cryptsetup open /dev/sda2 cryptroot
mount /dev/mapper/cryptroot /mnt

mkfs.fat -F32 /dev/sda1
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot

pacstrap /mnt base

genfstab -U /mnt >> /mnt/etc/fstab

# Save the /dev/sda2 UUID from:
blkid

arch-chroot /mnt
ln -sf /usr/share/zoneinfo/Pacific/Auckland /etc/localtime
hwclock --systohc
nano /etc/locale.gen
	...
locale-gen
echo LANG=en_US.UTF-8 > /etc/locale.conf
echo arch123 > /etc/hostname

echo "127.0.0.1    localhost" >> /etc/hosts
echo "::1          localhost" >> /etc/hosts
echo "127.0.1.1    archgundam.localdomain archgundam" >> /etc/hosts

nano /etc/mkinitcpio.conf
...
HOOKS=(base udev autodetect keyboard keymap consolefont modconf block encrypt filesystems fsck)
...
mkinitcpio -p linux

passwd

nano /etc/pacman.d/mirrorlist
...

pacman -S networkmanager
systemctl enable NetworkManager

pacman -S grub efibootmgr ttf-dejavu intel-ucode
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=grub --boot-directory=/boot
nano /etc/default/grub
...
GRUB_CMDLINE_LINUX_DEFAULT="cryptdevice=UUID=abc12345-6789-abcd-1234-a1b2c3d4f5g6:cryptroot root=/dev/mapper/cryptroot"
GRUB_FONT=/boot/grub/fonts/DejaVuSansMono.pf2
GRUB_ENABLE_CRYPTODISK=y
...
grub-mkfont -o /boot/grub/fonts/DejaVuSansMono.pf2 -s 24 /usr/share/fonts/TTF/DejaVuSansMono.ttf
grub-mkconfig -o /boot/grub/grub.cfg

# Exit chroot
exit

# Reboot out of install disk
reboot
```

```bash
pacman -S sudo
useradd -m -G wheel travers
passwd travers
EDITOR=nano visudo /etc/sudoers
...
%wheel ALL=(ALL) ALL
...

# Swap file
# File must be at least as large as RAM for hibernate to work
fallocate -l 16G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
nano /etc/fstab
...
# Swap file
/swapfile	none	swap	defaults	0	0
...

echo "vm.swappiness=10" > /etc/sysctl.d/80-sysctl.conf

# Hibernate
nano /etc/mkinitcpio.conf
...
HOOKS=(base udev autodetect keyboard modconf block encrypt filesystems resume fsck)
...
mkinitcpio -p linux

# Get the first physical offset from:
filefrag -v /swapfile

nano /etc/default/grub
...
GRUB_CMDLINE_LINUX_DEFAULT="resume=/dev/mapper/cryptroot resume_offset=247808 cryptdevice=UUID=abc12345-6789-abcd-1234-a1b2c3d4f5g6:cryptroot root=/dev/mapper/cryptroot"
...
grub-mkconfig -o /boot/grub/grub.cfg

# Reboot to enable hibernate / resume
reboot

echo "fs.inotify.max_user_watches=262144" > /etc/sysctl.d/40-max-user-watches.conf
```
