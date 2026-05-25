# Artix Linux Install Guide

## Rules — Read Before Starting

- **Do `pacman -Syu` before installing anything.** No exceptions. Individual package installs on an unupdated system will cause library mismatches and broken services.
- **Do not remove GRUB before verifying Limine boots.**
- **Verify `/etc/fstab` has the EFI partition before touching the bootloader.**
- **Never run `pacman -Syu` with both Artix and Arch repos enabled simultaneously.** Artix runs slightly behind Arch; a full update with both active can pull in package versions that are ahead of what Artix's own repos provide, causing version skew (lib32-expat is a known example). If you need a package from Arch `[multilib]`, enable it, install the specific package, then disable it and run `pacman -Syy` to restore the index.
- Services are not auto-enabled on Artix (OpenRC). Every service must be explicitly added with `rc-update`.


## 1. First Boot

### Verify EFI partition is in fstab

```bash
cat /etc/fstab
```

There must be an entry mounting the EFI partition at `/boot/efi`. If it is missing, add it before proceeding:

```
UUID=<EFI-UUID>  /boot/efi  vfat  defaults  0 2
```

Get the UUID with:

```bash
blkid /dev/sdXN   # replace with your EFI partition device
```

### Full system update

Do this before installing or configuring anything:

```bash
sudo pacman -Syu
```

Reboot if the kernel was updated.


### Set Consolefont (1440p)

Configure Console font. For my 1440p, 22px looked as a good defatlt. When changing the font, it's best to keep it within ter-v* family. `n` is for "normal", `b` is for bold. `ter-v22b` is safe, 22px, bold font.
Optionally edit `/etc/conf.d/consolefont` (OpenRC specific) to modify the default:
```
consolefont="ter-v22b"

```
For OpenRC, need to enable the service manually. Then reboot for it to take effect. This should remove the Pacman warning too.
```bash
sudo rc-update add consolefont boot
```


## 2. UFW Firewall

Get the firewall up before any extended network activity.

```bash
sudo pacman -S ufw ufw-openrc
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw limit 22/tcp
sudo ufw enable
sudo rc-service ufw start
sudo rc-update add ufw default
```

Note: do not add `allow 80/tcp` or `allow 443/tcp` on a desktop — those are incoming rules for servers. Browser traffic uses established outgoing connections and does not need open incoming ports.


## 3. Chrony (Time Sync)

Correct time is required for SSL and git operations. Do this before cloning anything.

First, check whether Calamares installed any conflicting time service:

```bash
rc-status -a | grep -E "ntp|chrony|time"
```

If `ntpd` or `openntpd` appears, disable and remove it before proceeding:

```bash
sudo rc-update del ntpd default       # if present
sudo pacman -Rns ntp                  # if installed
```

Then install and start Chrony:

```bash
sudo pacman -S chrony chrony-openrc
sudo rc-update add chrony default
sudo rc-service chronyd start
sudo chronyc makestep
```

Verify:

```bash
chronyc tracking
```


## 4. Limine Bootloader

### Install Limine

```bash
sudo pacman -S limine
```

### Copy EFI binary

```bash
sudo mkdir -p /boot/efi/EFI/limine
sudo cp /usr/share/limine/BOOTX64.EFI /boot/efi/EFI/limine/liminex64.efi
```

### Register with UEFI

```bash
sudo efibootmgr --create --disk /dev/nvme1n1 --part 2 --loader "EFI/limine/liminex64.efi" --label "Limine"

```

### Create limine.conf on the EFI partition

`/boot/efi/limine.conf` — example entry for btrfs with `@` subvolume and OpenRC init:

```
timeout: 5

/Artix Linux
    protocol: linux
    kernel_path: boot:///EFI/Linux/vmlinuz-linux
    kernel_cmdline: root=UUID=<ROOT-UUID> rootflags=subvol=@ rootfstype=btrfs rw init=/usr/bin/openrc-init quiet
    module_path: boot:///EFI/Linux/amd-ucode.img
    module_path: boot:///EFI/Linux/initramfs-linux.img
```

### Copy initial kernel files to ESP

```bash
sudo mkdir -p /boot/efi/EFI/Linux
sudo cp /boot/vmlinuz-linux /boot/efi/EFI/Linux/
sudo cp /boot/initramfs-linux.img /boot/efi/EFI/Linux/
[[ -f /boot/amd-ucode.img ]]   && sudo cp /boot/amd-ucode.img   /boot/efi/EFI/Linux/
[[ -f /boot/intel-ucode.img ]] && sudo cp /boot/intel-ucode.img /boot/efi/EFI/Linux/
```

### Verify Limine boots

Reboot. Confirm the Limine menu appears and the system boots successfully before proceeding.

### Remove GRUB and os-prober

Only after confirming Limine works. `os-prober` is a GRUB companion with no role after GRUB is removed — Limine has its own chainload support if dual-boot via Limine is ever needed.

```bash
sudo pacman -Rns grub artix-grub-theme os-prober
```


## 5. Timeshift Snapshot — Checkpoint 1

Clean system, Limine confirmed, GRUB gone. Take a snapshot before adding complexity.

```bash
sudo pacman -S timeshift
```

Open the Timeshift GUI to configure before taking the first snapshot:
- Snapshot type: **btrfs** (not rsync)
- Include subvolumes: `@` (root) and `@home` at minimum

Then take the baseline snapshot:

```bash
sudo timeshift --create --comments "Post-Limine baseline" --tags O
```


## 6. Clone Config Repository

```bash
mkdir -p ~/git
cd ~/git
git clone https://github.com/kbolshakov/artix-config.git
```

Don't do `git clone` for repos that you want to push to. Instead follow the auth steps, create a floder, do `git init`, then pull and push.
The repo directory structure mirrors the filesystem root — `etc/tlp.conf` belongs at `/etc/tlp.conf`, `usr/local/bin/update-esp-kernels` at `/usr/local/bin/`, and so on. Copy commands in subsequent sections use filenames only; the source path follows from the repo structure.

See `GIT-CMDS.md` for branch and remote setup.


## 7. ESP Sync Hook

Keeps the EFI partition in sync with `/boot/` after kernel updates.

### Install the sync scripts

```bash
sudo cp update-esp-kernels /usr/local/bin/
sudo cp mkinitcpio-sync /usr/local/bin/      # manual mkinitcpio wrapper
sudo chmod +x /usr/local/bin/update-esp-kernels
sudo chmod +x /usr/local/bin/mkinitcpio-sync
```

The `update-esp-kernels` script must include a mount guard and per-kernel conditional copies:

```bash
mountpoint -q /boot/efi || { echo "[ESP SYNC] /boot/efi not mounted, aborting"; exit 1; }

[[ -f /boot/vmlinuz-linux ]]     && rsync -av /boot/vmlinuz-linux     /boot/initramfs-linux.img     "$ESP/"
[[ -f /boot/vmlinuz-linux-lts ]] && rsync -av /boot/vmlinuz-linux-lts /boot/initramfs-linux-lts.img "$ESP/"
[[ -f /boot/vmlinuz-linux-zen ]] && rsync -av /boot/vmlinuz-linux-zen /boot/initramfs-linux-zen.img "$ESP/"
[[ -f /boot/amd-ucode.img ]]     && cp -v /boot/amd-ucode.img   "$ESP/"
[[ -f /boot/intel-ucode.img ]]   && cp -v /boot/intel-ucode.img "$ESP/"
```

### Install the pacman hook

```bash
sudo mkdir -p /etc/pacman.d/hooks
sudo cp 92-esp-sync.hook /etc/pacman.d/hooks/
```


## 8. iwd (WiFi Backend)

iwd is faster and cleaner than wpa_supplicant when managed through NetworkManager.

```bash
sudo pacman -S iwd iwd-openrc
sudo rc-update add iwd default
sudo rc-update del wpa_supplicant default
```

Edit `/etc/NetworkManager/NetworkManager.conf`:

```ini
[device]
wifi.backend=iwd
```

```bash
sudo rc-service iwd start
sudo rc-service NetworkManager restart
```

After that chrony, netmount (maybe something else) may be stopped and need a restart.


## 9. TLP (Power Management)

```bash
sudo pacman -S tlp tlp-openrc
sudo rc-update add tlp default
sudo rc-service tlp start
```

Copy config from the repo:

```bash
sudo cp tlp.conf /etc/tlp.conf
sudo rc-service tlp restart
```


## 11. Enable lib32 Repo

Required for Steam and 32-bit applications. Artix's own `[lib32]` repo is the right choice for a permanent addition — it stays in sync with Artix's main repos.

Edit `/etc/pacman.conf` and uncomment or add:

```ini
[lib32]
Include = /etc/pacman.d/artix-mirrorlist
```

Then refresh:

```bash
sudo pacman -Syu
```

If a specific package is only available in Arch's `[multilib]`, enable it temporarily, install the package, then disable it and run `pacman -Syy`. Do not run a full `-Syu` with both repos active.


## 12. Configuration Files

Copy from the cloned repo or set up symlinks. Key locations:

- **KDE Plasma**: `.config/plasma-org.kde.plasma.desktop-appletsrc`
- **Konsole**: `.config/konsolerc`, `.local/share/konsole/Profile1.profile`
- **Brave Browser**: `.config/BraveSoftware/Brave-Browser/Default/Preferences` — note: Brave is AUR-only, Yay must be installed before Brave can be installed (see section 14)
- **Flatpak**: `sudo pacman -S flatpak`, then: `flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo`


## 13. General Packages

TBD — expand with full package list.


## 14. Yay (AUR Helper)

Install when an AUR package is needed. Brave is AUR-only, so this is a prerequisite for it.

```bash
sudo pacman -S --needed base-devel git
cd ~/Downloads
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```


## 15. Steam

See `STEAM.md`. Requires lib32 repo to be enabled (section 11). Treat as a separate effort — the base system is fully functional without it.
