# Initial install

## 1. Install Artix: Plasma ISO
This used Calamares, should be self-explanatory.

Install Yay while at it:
```fish
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
```fish
pacman -Rns ntp ntp-openrc
pacman -S chrony chrony-openrc
```
This can be done later, just skip chrony and refer to Section 5.

## 4. Configure Limine
Once the package is installed, copy:
```fish
cp /usr/share/limine/BOOTX64.EFI /boot/efi/EFI/limine/liminex64.efi
```
Then replicate the folder structure and run `mkinitcpio-sync`.

## 5. Important Services (ufw, chrony, others)
### (1) Install: ufw, ufw-openrc
#### Configure the rules
```fish
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw limit 22/tcp
sudo ufw enable
```

#### Enable and start the service
```fish
sudo rc-service ufw start
sudo rc-update add ufw default
     rc-service ufw status
     rc-status -a
```

#### Config status check
```fish
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
```fish
sudo rc-service ntpd stop
sudo rc-update del ntpd default
```

Confirm it's gone:
```fish
rc-status | grep ntpd
ps aux | grep ntpd
```

Then install:
```fish
sudo pacman -S chrony chrony-openrc
```

#### Enable, start, sync
Enable and start the service:
```fish
sudo rc-update add chrony default
sudo rc-service chrony start
```
chrony’s config is in `/etc/chrony.conf`

Initial sync:
```fish
sudo chronyc makestep
```

Lastly, check the status:
```fish
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

#### Check the rest (other)
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

This is the way it works in Artix.
Make sure `kwallet-pam` is installed.
```fish
sudo pacman -Ss kwallet-pam
```
Modity this file: `/etc/pam.d/system-login` as it is in the repo.
Then go to KWalletManager and change the password of 'kdewallet' to match your login one.
The name of the wallet is important. Chenge the password even if it's already the same.
Logout and log back in. Brave should now load without asking for KWallet password.

### (3) Flatpak
Install Flatpak:
```fish
sudo pacman -S flatpak
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

Optionally, add KDE GUI for it:
```fish
sudo pacman -S discover  # slow, but native, may still be wirth keeping for fwupd
sudo pacman -S bazaar    # Gnome-like (GTK), but modern and nice, must be installed through Pacman, not Flatpak!
```
Logout and log back in.

### (4) Steam and Gamescope
There is a whole file dedicated to this.
See [STEAM.md](STEAM.md).
Can be done at any time, once the core system and services are ready.



## 7. Future: TBD
