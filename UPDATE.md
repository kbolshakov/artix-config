# This is a file to keep track of what may need to be done to keep this up to date

## 1. Package lists, keep current:

Keep in mind that yay is special, installed manually. May be scripted later.
```bash
pacman -Qqen > pkglist.txt        # pacman package list
pacman -Qqm  > pkglist-aur.txt    # AUR (yay) package list
```


## 2. OpenRC Services, keep current:
This is the authoritative source, all services are there with their assigned runlevels.
```bash
rc-update show -a > openrc-services.txt
```

## 3. Consider a script to update the settings later.
TBD