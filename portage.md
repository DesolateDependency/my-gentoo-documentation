# My Gentoo cheatsheet
List all packages that are installed on the system.
```
ls /var/db/pkg/*
```
List all packages that i installed.
```
cat /var/lib/portage/world
```
Do changes to config files.
```
dispatch-conf
```
Install new package.
```
emerge -av <package-name>
```
Uninstall package.
```
emerge -avc <package-name>
```
Sync repos.
```
emerge --sync
```
Update all packages in @world.
```
emerge -avuDU @world
```
Uninstall unused dependencies.
```
emerge -ac
```
