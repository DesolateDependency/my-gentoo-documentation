# My cheatsheet
Check if swap is mounted.
```
$ swapon --show
```
Execute as root.
```
$ doas <command>
```
Execute as other user.
```
$ doas -u <username> <command>
```
Do changes to config files.
```
$ dispatch-conf
```
List all packages that i installed.
```
# cat /var/lib/portage/world
```
List all packages that are installed on the system.
```
$ ls /var/db/pkg/*
```
Install new package.
```
$ emerge -av <package-name>
```
Uninstall package.
```
$ emerge -avc <package-name>
```
Sync repos.
```
$ emerge --sync
```
Update all packages in @world.
```
$ emerge -avuDU @world
```
Uninstall unused dependencies.
```
$ emerge -ac
```
Check Qtile config.
```
# python3 ~/.config/qtile/config.py
```
