# Add Arch Repos support
```fish
sudo pacman -S artix-archlinux-support
```
This will print:
```
:: Running post-transaction hooks...
(1/1) Show archlinux help...
==> Add the arch repos in pacman.conf:

#[extra-testing]
#Include = /etc/pacman.d/mirrorlist-arch

[extra]
Include = /etc/pacman.d/mirrorlist-arch

#[multilib-testing]
#Include = /etc/pacman.d/mirrorlist-arch

#[multilib]
#Include = /etc/pacman.d/mirrorlist-arch

==> run: 'pacman-key --populate archlinux'
```

Need to add these to pacman.conf manually. Then run:
```fish
sudo pacman-key --populate archlinux
```

May be a good idea to re-run reflector with new repos added:
```fish
sudo reflector --sort rate --save /etc/pacman.d/mirrorlist
```

After that, sync and update:
```fish
sudo pacman -Syu
```

## Install Steam
```fish
sudo pacman -S multilib/steam
```
When prompted, the only choice needed should be `lib32-vulkan-radeon` for AMD GPU.
For others, read pacman descriptions and/or consult other resources.

## Add controller support
Install udev:
```fish
yay -S game-devices-udev
```
Then reboot. After reboot, check:
```fish
lsmod | grep uinput
ll /dev/uinput
```
The controller was detected in Steam after that.

## Install Gamescope
The Gamescope package itself:
```fish
sudo pacman -S world/gamescope
```
Then, the Gamescope session from AUR. This is the one used in EOS setup, customized.
```fish
yay -S aur/gamescope-session-steam-sk-git
```

## Configure Gamescope sessions
### Real launcher script
Located in:
```
/usr/share/gamescope-session-plus/gamescope-session-plus
```
Modify "USR_MONITOR" from `/tmp`.

### Wrapper scripts
This is what is called from .desktop session. We need 3 of them:
```
/usr/bin/gamescope-session-plus
/usr/bin/gamescope-session-plus-dp
/usr/bin/gamescope-session-plus-hdmi
```
Optional 4th, was used for debug logging:
```
/usr/bin/gamescope-session-plus-debug
```

### SDDM session files
These are actrual session configs, they show up in SDDM and call the wrapper scripts.
One session per script (SDDM limitaion, but clean):
```
/usr/share/wayland-sessions/gamescope-session-steam.desktop
/usr/share/wayland-sessions/gamescope-session-steam-dp.desktop
/usr/share/wayland-sessions/gamescope-session-steam-hdmi.desktop
```
Files are backed up exaclty.
Video may still lag a few seconds, audio from Startup is quick.