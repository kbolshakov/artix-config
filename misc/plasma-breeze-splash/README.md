## Artix - Plasma Breeze custom splash
This is a modified Breeze splash with Artix logo and spinner from "Arch" splash.
This can be reproduced or modified by grabbing another logo from `/usr/share/icons/artix/`,
copying it to `/usr/share/plasma/look-and-feel/org.kde.breeze.desktop/contents/splash/images/`
and renaming is as `plasma.svgz`.

The spinner `busywidget.svgz` is taken from the "Arch Simple Blue KDE 6" splash install.
Just use `fd` once the package is installed.

Preview logo:

![plasma](https://github.com/kbolshakov/artix-config/misc/plasma-breeze-splash/.preview/plasma.svg)


Preview spinner:

![busywidget](https://github.com/kbolshakov/artix-config/misc/plasma-breeze-splash/images/busywidget.svgz)

Lastly, the size is modified by a multiplyer of the 'logo' in `/usr/share/plasma/look-and-feel/org.kde.breeze.desktop/contents/splash/Splash.qml`.

For simplicity, just copy these files to correcponding folders.