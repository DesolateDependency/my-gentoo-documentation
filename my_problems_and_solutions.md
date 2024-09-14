# My problems and solutions
### Swap is not getting mounted with OpenRC init at boot.
**Problem:** \
The OpenRC boot log shows the following error and the swap is not mounting at boot. Mounting afterwards manually with `swapon` works.
> swapon: cannot open /swap/swapfile: No such file or directory 

\
**Solution:** \
I had to setup OpenRC so it can use swapfiles which are not on the root filesystem. To do that i had to adjust the `/etc/conf.d/swap` file like this.

_/mnt/gentoo/etc/conf.d/swap_
```
...
rc_before="!localmount"
...
rc_need="localmount"
...
```

### LUKS encryption passphrase screen has the wrong keyboard layout
**Problem:** \
The LUKS encryption passphrase screen uses a english keyboard layout by default. The set keymap is set with the OpenRC boot later.

\
**Solution:** \
Dracut wants to have a vconsole.conf file with the keymap. (defaults to en) \

_/etc/vconsole.conf_
```
KEYMAP=de-latin1
```
\
Don't forget to regenerate the the initramfs.
```
# doas dracut --force
```

### Can't add seatd to rc-services
**Problem:** \
I am not able to add seatd to the rc-services to make it start at boot with `rc-update add seatd default`.

\
**Solution:** \
The _builtin_ and _server_ USE-flags have to be added to the seatd package.

_/etc/portage/package.use/seatd
```
...
sys-auth/seatd builtin server
```
\
And then it has to be reinstalled.
```
# emerge --ask --verbose --update --deep --newuse @world
```

### Dracut can't find cryptroot after luks password
**Problem:** \
Dracut can't find cryptroot after typing in the luks password. This happened after a fresh installation.

\
**Solution:** \
Initramfs was not setup correctly. We were missing a dracut configuration and a crypttab.
First create a configuration in _/etc/dracut.conf.d/_.
```
# mkdir /etc/dracut.conf.d
# nvim /etc/dracut.conf.d/encryption.conf
```

_/etc/dracut.conf.d/encryption.conf_
```
hostonly="yes"
add_dracutmodules+=" crypt dm rootfs-block "
kernel_cmdline+=" root=UUID=<uuid-of-cryptroot> rd.luks.uuid=<uuid-of-root-partition> "
```
\
We want to check which kernel we are running on our future machine.
```
# ls /lib/modules
```
\
The initramfs has to be build like this.
````
dracut --force --kver <kernel-version>
````
\
Now we have to create a crypttab file.

_/etc/crypttab_
```
cryptroot UUID=<uuid-of-encrypted-root-partition> none luks
```

### Root is read-only during boot
**Problem:** \
Root partition doesn't get remounted as read-write after init, which creates many errors during bootup.

\
**Solution:** \
There was a mistake in the fstab file. This lead to root not remounting the root partition and thus leading to errors on bootup.
