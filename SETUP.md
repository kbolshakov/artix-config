# Initial install

## 1. Install Artix: Plasma ISO
This used Calamares, should be self-explanatory.

Install Yay while at it:
```bash
sudo pacman -S --needed base-devel git
cd Downloads
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```
Delete the `yay` folder after.

## 2. Clone repo
Refer to [GIT-CMDS.md](GIT-CMDS.md) for relevant commands.

## 3. Install packages from pkglist
Explicitly remove NTP and install chrony, cannot have both at the same time.
```bash
pacman -Rns ntp ntp-openrc
pacman -S chrony chrony-openrc
```
This can be done later, just skip chrony and refer to Section 5.

## 4. Configure Limine
Once the package is installed, copy:
```bash
cp /usr/share/limine/BOOTX64.EFI /boot/efi/EFI/limine/liminex64.efi
```
Then replicate the folder structure and run `mkinitcpio-sync`.

## 5. Important Services (ufw, chrony, others)
### (1) Install: ufw, ufw-openrc
#### Configure the rules
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw limit 22/tcp
sudo ufw enable
```

#### Enable and start the service
```bash
sudo rc-service ufw start
sudo rc-update add ufw default
     rc-service ufw status
     rc-status -a
```

#### Config status check
```bash
sudo ufw status verbose
```
Output should look like this:
```fish
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     LIMIT IN    Anywhere                  
80/tcp                     ALLOW IN    Anywhere                  
443/tcp                    ALLOW IN    Anywhere                  
22/tcp (v6)                LIMIT IN    Anywhere (v6)             
80/tcp (v6)                ALLOW IN    Anywhere (v6)             
443/tcp (v6)               ALLOW IN    Anywhere (v6) 
```

### (2) Install: chrony, chrony-openrc
#### Ensure NTP is disabled and removed
First, remove NTP:
```bash
sudo rc-service ntpd stop
sudo rc-update del ntpd default
```

Confirm it's gone:
```bash
rc-status | grep ntpd
ps aux | grep ntpd
```

Then install:
```bash
sudo pacman -S chrony chrony-openrc
```

#### Enable, start, sync
Enable and start the service:
```bash
sudo rc-update add chrony default
sudo rc-service chrony start
```
chrony’s config is in `/etc/chrony.conf`

Initial sync:
```bash
sudo chronyc makestep
```

Lastly, check the status:
```bash
chronyc tracking
```

Ideally, we want to see 3 sources (Stratum) and a small offset. Example at the time of writing:
```fish
Reference ID    : 8AC5A436 (xlrsecurity.com)
Stratum         : 3
Ref time (UTC)  : Mon Apr 27 23:22:36 2026
System time     : 0.000900050 seconds slow of NTP time
Last offset     : -0.000849867 seconds
RMS offset      : 0.000515330 seconds
Frequency       : 65.297 ppm slow
Residual freq   : -0.048 ppm
Skew            : 1.147 ppm
Root delay      : 0.007765875 seconds
Root dispersion : 0.005940241 seconds
Update interval : 518.9 seconds
Leap status     : Normal
```
Now just let it run, references can change.

### (3) Install: iwd, iwd-openrc
#### Replacing wpa_supplicant in NetworkManager
First, install iwd:
```bash
sudo pacman -S iwd iwd-openrc
```

Disable wpa_supplicant, add `iwd` to default and start the service:
```bash
sudo rc-service wpa_supplicant stop
sudo rc-update add iwd default
sudo rc-service iwd start
```

Edit `/etc/NetworkManager/NetworkManager.conf` file to delegate wifi to `iwd`. There is likely nothing in that file except comments.
```bash
[device]
wifi.backend=iwd

```

Restart NetworkManager, check that `iwd` is working with wifi:
```bash
sudo rc-service NetworkManager restart
rc-service iwd status
iwctl station list
```

Reboot. Ideally, twice: the 1st time it can take a bit longer. The 2nd time it should be faster.

### Check the rest (other)
Most of the important services should be enabled, but it's good to audit the result using `openrc-services.txt` file.

## 6. Configs
### (1) KDE Plasma
This is a tricky one, may not work. Even if it does, som parts need to be modified manually to be safe.

#### Panel, widgets, clock
Replace this file for the bar/panel and widgets + clock settings:
`.config/plasma-org.kde.plasma.desktop-appletsrc`

#### Konsole
Konsole preferences are split.
General settings, such as buttons and tabs:
`.config/konsolerc`

Profile settings, such as font and color:
`.local/share/konsole/Profile1.profile`

### (2) Brave Browser + KDE Wallet
This only needs one file that saves the important preferences, including fonts:
`.config/BraveSoftware/Brave-Browser/Default/Preferences`

KDE Wallet is a good way to store passwords and keys.
Having it unlock by default has been a tricky thing for me in the past.
Decided to leave this extra step that requires manual password entry upon boot.
This is better than no-password for the wallet, safer for a portable SSD install, and saves troubleshooting time.

### (3) Flatpak
Install Flatpak:
```bash
sudo pacman -S flatpak
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

Optionally, add KDE GUI for it:
```bash
sudo pacman -S discover  # slow, but native, may still be wirth keeping for fwupd
sudo pacman -S bazaar    # Gnome-like (GTK), but modern and nice, must be installed through Pacman, not Flatpak!
```
Logout and log back in.

### (4) Steam and Gamescope
There is a whole file dedicated to this.
See [STEAM.md](STEAM.md).
Can be done at any time, once the core system and services are ready.



## 7. Future: TBD
