## Initial install

1. Install Artix: Plasma ISO
2. Clone repo
3. Install packages from pkglist
	pacman -Rns ntp ntp-openrc
	pacman -S chrony chrony-openrc
	do the rest
4. Configure Limine
	cp /usr/share/limine/BOOTX64.EFI /boot/efi/EFI/limine/liminex64.efi
	do the rest (folder structure)
5. Copy other configs (ufw, other...)
6. Enable services
7. (future opt) Run scripts
