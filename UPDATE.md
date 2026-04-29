# This is a file to keep track of what may need to be done to keep this up to date

## 1. Package lists, keep current:
```bash
pacman -Qqen > pkglist.txt        # pacman package list
pacman -Qqm  > pkglist-aur.txt    # AUR (yay) package list
```
Keep in mind that yay is special, and AUR is a subset of Pacman here.

## 2. Consider a script to update the settings later.