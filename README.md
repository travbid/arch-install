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

pacstrap /mnt base linux

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

pacman -Sg gnome
# gnome baobab
# gnome cheese
# gnome eog
# gnome epiphany
# gnome evince
# gnome file-roller
# gnome gdm
# gnome gedit
# gnome gnome-backgrounds
# gnome gnome-books
# gnome gnome-calculator
# gnome gnome-calendar
# gnome gnome-characters
# gnome gnome-clocks
# gnome gnome-color-manager
# gnome gnome-contacts
# gnome gnome-control-center
# gnome gnome-dictionary
# gnome gnome-disk-utility
# gnome gnome-documents
# gnome gnome-font-viewer
# gnome gnome-getting-started-docs
# gnome gnome-keyring
# gnome gnome-logs
# gnome gnome-maps
# gnome gnome-menus
# gnome gnome-music
# gnome gnome-photos
# gnome gnome-remote-desktop
# gnome gnome-screenshot
# gnome gnome-session
# gnome gnome-settings-daemon
# gnome gnome-shell
# gnome gnome-shell-extensions
# gnome gnome-system-monitor
# gnome gnome-terminal
# gnome gnome-themes-extra
# gnome gnome-todo
# gnome gnome-user-docs
# gnome gnome-user-share
# gnome gnome-video-effects
# gnome gnome-weather
# gnome grilo-plugins
# gnome gvfs
# gnome gvfs-afc
# gnome gvfs-goa
# gnome gvfs-google
# gnome gvfs-gphoto2
# gnome gvfs-mtp
# gnome gvfs-nfs
# gnome gvfs-smb
# gnome mousetweaks
# gnome mutter
# gnome nautilus
# gnome networkmanager
# gnome orca
# gnome rygel
# gnome sushi
# gnome totem
# gnome tracker
# gnome tracker-miners
# gnome vino
# gnome xdg-user-dirs-gtk
# gnome yelp
# gnome gnome-boxes
# gnome gnome-software
# gnome simple-scan

pacman -S baobab eog evince file-roller gdm gedit gnome-backgrounds gnome-calculator gnome-characters gnome-clocks gnome-control-center gnome-disk-utility gnome-font-viewer gnome-session gnome-shell gnome-shell-extensions gnome-system-monitor gnome-terminal gnome-themes-extra mutter networkmanager simple-scan

systemctl enable gdm

reboot

pacman -Sg gnome-extra
pacman -S gnome-tweaks gnome-usage
pacman -S thunar sakura vlc

# Log-in screen logo
# Copy arch-crystal-white.png to /usr/share/pixmaps
echo "[org/gnome/login-screen]" > /etc/dconf/db/gdm.d/01-logo
echo "  logo='/usr/share/pixmaps/arch-crystal-white.png'" >> /etc/dconf/db/gdm.d/01-logo
```
