# My problems and solutions
### Swap is not getting mounted with OpenRC init at boot.
**Problem:**
The OpenRC boot log shows the following error and the swap is not mounting at boot. Mounting afterwards manually with `swapon` works.
> swapon: cannot open /swap/swapfile: No such file or directory

**Solution:**
I had to setup OpenRC so it can use swapfiles which are not on the root filesystem. To do that i had to adjust the `/etc/conf.d/swap` file like this.

_/mnt/gentoo/etc/conf.d/swap_
```
...
rc_before="!localmount"
...
rc_need="netmount"
...
```
