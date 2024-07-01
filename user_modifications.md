# User modifications
## Rename user
Renaming username and home directory. This also deletes the faillock file for the old name. A new one will be created automatically with the new name.
```
# usermod -d /home/<new-name> -m -l <new-name> <old-name>
# rm /run/faillock/<old-name>
```
> [!NOTE]
> This will not rename the mail spool.
