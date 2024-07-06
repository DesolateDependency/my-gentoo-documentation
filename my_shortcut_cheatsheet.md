# My shortcut cheatsheet
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
Install new package.
```
$ doas emerge --ask <package_name>
```
Do changes to config files.
```
$ doas dispatch-conf
```
