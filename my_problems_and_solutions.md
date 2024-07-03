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
rc_need="netmount"
...
```

### LUKS encryption passphrase screen has the wrong keyboard layout
**Problem:** \
The LUKS encryption passphrase screen uses a english keyboard layout by default. The set keymap is set with the OpenRC boot later.

\
**Solution:** \
The right keyboard layout has to be set in the grub boot parameters to change the grub keymap for the LUKS encryption. \
A list of all available keymaps can be found here.
```
# nano /usr/share/genkernel/keymaps
```
The listed keymaps, without the '.map' ending, can be used in the configs.

_/etc/default/grub_
```
...
GRUB_CMDLINE_LINUX_DEFAULT="... keymap=de"
```
\
Don't forget to regenerate the grub boot option configuration after editing the grub file.
```
# grub-mkconfig -o /boot/grub/grub.cfg
```
