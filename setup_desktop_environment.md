# Setup of my desktop environment

## Install display manager
Add accepted keywork for tuigreet so in can be installed.

_/etc/portage/package.accept_keywords/tuigreet_
```
gui-apps/tuigreet ~amd64
```
\
Now install display-manager-init, greetd and tuigreet.
```
# doas emerge -av gui-libs/greetd gui-apps/tuigreet
```
\
Add display-manager-init zu OpenRC.
```
$ rc-update add display-manager default
```
\
Configure the display-manager.

_/etc/conf.d/display-manager_
```
CHECKVT=7
DISPLAYMANAGER="greetd"
```
\
Configure greetd.

_/etc/greetd/config.toml_
```
[terminal]
vt = 7
 
[default_session]
command = "tuigreet --cmd /bin/bash -t --remember-session"
user = "greetd"
```
\
Modify greetd user.
```
$ usermod greetd -aG video
$ usermod greetd -aG input
$ usermod greetd -aG seat
```

## Install window manager
Add USE flags.

_/etc/portage/package.use/qtile_
```
x11-libs/cairo X glib opengl svg
```
\
Add qtiles testing version to accept_keywords.

_/etc/portage/package.accept_keywords/qtile_
```
x11-wm/qtile ~amd64
```
\
Install Qtile.
```
$ emerge -av x11-wm/qtile
```
\
Set keyboard layout.

_/etc/environment_
```
XKB_DEFAULT_LAYOUT=de
```

## Install terminal emulator
Install foot.
```
$ emerge -av gui-apps/foot
```
\
Set foot as the default terminal in qtile.
