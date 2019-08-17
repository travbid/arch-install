## Fix Thunar on Wayland
- ```sudo nano /usr/share/applications/Thunar.desktop```
- Change the `Exec` line to: ```Exec=env GDK_BACKEND=x11 thunar %F```

## Increase Sakura terminal size
- Open `~/.local/share/applications/sakura.desktop`
- Change the `Exec` line to: ```Exec=sakura -c 120 -r 36```
