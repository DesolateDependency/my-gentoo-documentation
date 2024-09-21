# Installation guide (OpenRC, Btrfs, LUKS, Dist. Kernel, Swap, minimal/desktop)
> [!IMPORTANT]
> This setup is heavily adjusted to my own needs and may not mention important things for your specific usecase. I recommend following the [Gentoo handbook](https://wiki.gentoo.org/wiki/Handbook:AMD64/Full/Installation) and only use my guide as additional information.

## Research sources

I used the following sources to collect information:
- Gentoo amd64 handbook [Full Install guide](https://wiki.gentoo.org/wiki/Handbook:AMD64/Full/Installation)
- [LTO article](https://wiki.gentoo.org/wiki/LTO) and [Encryption article](https://wiki.gentoo.org/wiki/Full_Disk_Encryption_From_Scratch) in Gentoo wiki 
- [Btrfs article](https://wiki.archlinux.org/title/Btrfs) in Arch wiki 
- Youtube video _([The Best OS: Install Gentoo Linux on an encrypted btrfs root with optional Sway WM.](https://youtu.be/t0nPxDlFL2I?si=FyU1lBa7gS3blI2J))_ by libreisaac 
- Youtube video _([Gentoo: A ℂ𝕠𝕞𝕗𝕪 Install Guide](https://youtu.be/J7W9MItUSGw?si=fOtzomQ0E5WZ6-Bi))_ by Denshivideo 
- My dying braincell

## Summary

1. [Get image for installation](#1-get-image-for-installation) 
2. [Verify the downloaded files (Windows)](#2-verify-the-downloaded-files-windows)
3. [Writing the boot media (Windows)](#3-writing-the-boot-media-windows)
4. [Boot into life environment](#4-boot-into-life-environment)
___
5. [Preparation](#5-preparation)
6. [Testing the network](#6-testing-the-network)
7. [Connect to wifi (if not connected over ethernet)](#7-connect-to-wifi-if-not-connected-over-ethernet)
8. [Sync network time](#8-sync-network-time)
9. [Partitioning](#9-partitioning)
10. [Encrypting root](#10-encrypting-root)
11. [Formating & mounting partitions](#11-formating--mounting-partitions)
12. [Create and mount subvolumes](#12-create-and-mount-subvolumes)
13. [Create swap](#13-create-swap)
14. [Downloading, varifying and unpacking stage3](#14-downloading-varifying-and-unpacking-stage3)
15. [Adjust swap settings](#15-adjust-swap-settings)
16. [Set locales](#16-set-locales)
17. [Check clock](#17-check-clock)
18. [Set keymap](#18-set-keymap)
19. [Set timezone](#19-set-timezone)
20. [Create FS tables](#20-create-fs-tables)
21. [Setup initramfs](#21-setup-initramfs)
22. [Setup grub](#22-setup-grub)
23. [Setup portage](#23-setup-portage)
24. [Change into root](#24-change-into-root)
25. [[If interrupted]](#25-if-interrupted)
___
26. [Updating the Gentoo ebuild repository](#26-updating-the-gentoo-ebuild-repository)
27. [Select Profile](#27-select-profile)
28. [Applying localization configurations](#28-applying-localization-configurations)
29. [Set cpu flags](#29-set-cpu-flags)
30. [Updating the @world set](#30-updating-the-world-set)
31. [Install core packages](#31-install-core-packages)
32. [Configure doas](#32-configure-doas)
33. [Configuring services](#33-configuring-services)
34. [Managing user accounts](#34-managing-user-accounts)
35. [Compile kernel](#35-compile-kernel)
36. [Install bootloader](#36-install-bootloader)
37. [Rebooting](#37-rebooting)
___
38. [Lock root password](#38-lock-root-password)
39. [Connect Wifi](#39-connect-wifi)

## Install
### 1. Get image for installation
We will use the minimal gentoo install CD. To do that we have to download the iso image first by following these steps:
1. Connect to the mirror, typically using a local one found at [Gentoo source mirrors](https://www.gentoo.org/downloads/mirrors/).
2. Navigate to the _releases/_ directory.
3. Select the directory for the relevant target architecture (such as _amd64/_).
4. Select the _autobuilds/_ directory.
5. For _amd64_ and _x86_ architectures select either the _current-install-amd64-minimal/_ or _current-install-x86-minimal/_ directory (respectively). For all other architectures navigate to the _current-iso/_ directory.
6. Downlad the _.iso_ and the corresponding _.iso.asc_ file.

### 2. Verify the downloaded files (Windows)
> [!NOTE]
> This step is optional, but recommended to check if the file is corrupt or malicious.

On Windows you are able to do that with the help of [GPG4Win](https://www.gpg4win.org/).During the installation you can uncheck any additional software that can be installed. After installing it you have to run this command in the terminal at the location of your _.iso_ and _.iso.asc_ files.
```
gpg --verify install-amd64-minimal-******.iso.asc 
```
If it says that there is a good signature from _"Gentoo Linux Release Engineering (Automated Weekly Release Key) <releng@gentoo.org>"_ it is good.

After comparing the fingerprint in the output with the fingerprint on the [Gentoo signatures page](https://www.gentoo.org/downloads/signatures/) we can be sure that the file is save.

### 3. Writing the boot media (Windows)
I used the [Rufus](https://rufus.ie/en/) software to make an bootable USB stick.

### 4. Boot into life environment
To boot into the live environment you have to plug the stick into the new system befor turning it on. After changing the boot sequenz to the USB stick you will be able to boot into it after a reboot. Also make sure that secure boot is disabled. And if you had Windows befor make sure to disable bitlocker.

### 5. Preparation
After booting into the minimal install on the boot device it promts for a keyboard layout. If chosen "de" a few of the keys are still wrong. So i set the keymap manualy.
```
# loadkeys de-latin1
```

### 6. Testing the network
We want to check if we have a working internet connection because that is needed for the installation. First we see if we can ping. It should just work if the laptop/pc is connected via a ethernet cable.
```
# ping archlinux.org
```
\
Additional information can be gathered with the following command.
```
# ip route
# ip link
# ip address
```

### 7. Connect to wifi (if not connected over ethernet)
To connect to a wifi network we will use the `wpa_cli` tool. Before we can use it we have to set it up.
To do that we have to create a WPA supplicant configuration file.

_/etc/wpa_supplicant/wpa_supplicant.conf_
```
ctrl_interface=/run/wpa_supplicant
update_config=1
```
\
Now you have to start the wpa_supplicant service.
```
# rc-service wpa_supplicant start
```
\
After the setup is done we can start the wpa_cli application.
```
# wpa_cli
```
\
To setup the connection you have to run the following commands inside of this application and it should look similar like this.
```
> scan
OK
> scan_results
bssid / frequency / signal level / flags / ssid
<...> <...> <...> <...>
> add_network
0
> set_network 0 ssid "<network_ssid>"
OK
> set_network 0 psk "<network_password>"
OK
> enable_network 0
OK
> save_config
OK
> quit
```
> [!NOTE]
> Logs like "<3>CTRL_EVENT_SCAN_STARTED" or "<3>CTRL_EVENT_NETWOR_NOT_FOUND" etc. can be ignored.

You can check if it worked in here.
```
# ip link
```
If the link for the wifi entry shows UP it is connected.

### 8. Sync network time
We will run chronyd to sync up our time.
```
# chronyd -q
```

### 9. Partitioning
To check what drives are currently connected we can call this command.
```
# lsblk
```
\
At the beginning we want to override all data that was on it bevor for safety reasons.
```
# dd if=/dev/urandom of=/dev/<name-of-drive> bs=4096 status=progress
```
> [!NOTE]
> This step is optional, but most of the time best practice, as long as you don't do it too often. This overwrites the whole disk so it wears it down alot.

\
We will use parted to partition our disks. Open it by typing this.
```
# parted /dev/<name-of-drive>
```
\
Make a gpt table.
```
mklabel gpt
```
\
Create partitions for boot and root.
```
mkpart boot fat32 0% 1G
mkpart root btrfs 1G 100%
```
\
Mark the boot partition as bootable.
```
set 1 boot on
```
\
To view the created partitions you can call this command.
```
p
```
\
Now we can close parted.
```
q
```

### 10. Encrypting root
First we have to encrypt our root and open it.
```
# cryptsetup luksFormat -s256 -c aes-xts-plain64 /dev/<name-of-root-partition>
# cryptsetup luksOpen /dev/<name-of-root-partition> cryptroot
```

### 11. Formating & mounting partitions
Lets format our partitions.
```
# mkfs.vfat -F 32 /dev/<name-of-boot-partition>
# mkfs.btrfs -L BTROOT /dev/mapper/cryptroot
```
\
Now we have to mount our root partitions.
```
# mkdir /mnt/root
# mount -t btrfs -o defaults,noatime,compress=lzo,autodefrag /dev/mapper/cryptroot /mnt/root/
```

### 12. Create and mount subvolumes
We can create subvolumes on the root partition to organize our filesystem.
```
# btrfs subvolume create /mnt/root/activeroot
# btrfs subvolume create /mnt/root/swap
# btrfs subvolume create /mnt/root/home
```
\
Afterwards we have to mount the subvolumes.
```
# mount -t btrfs -o defaults,noatime,compress=lzo,autodefrag,subvol=activeroot /dev/mapper/cryptroot /mnt/gentoo/
# mkdir /mnt/gentoo/swap
# mount -t btrfs -o defaults,noatime,compress=lzo,autodefrag,subvol=swap /dev/mapper/cryptroot /mnt/gentoo/swap/
# mkdir /mnt/gentoo/home
# mount -t btrfs -o defaults,noatime,compress=lzo,autodefrag,subvol=home /dev/mapper/cryptroot /mnt/gentoo/home/
# mkdir /mnt/gentoo/boot
# mkdir /mnt/gentoo/efi
# mount /dev/<name-of-boot-partition> /mnt/gentoo/boot/
# mount /dev/<name-of-boot-partition> /mnt/gentoo/efi/
```

### 13. Create swap
Create swap.
```
# btrfs filesystem mkswapfile --size 16g --uuid clear /mnt/gentoo/swap/swapfile
```
\
Activate the swap file.
```
# swapon /mnt/gentoo/swap/swapfile
```

### 14. Downloading, varifying and unpacking stage3
Enter gentoos text based download page.
```
# links https://distfiles.gentoo.org/releases/amd64/autobuilds
```
1. Navigate to the latest timestamp directory.
2. Go to the "stage3-amd64-desktop-openrc-..." file
3. Pess d to download it
4. Got to the .asc file for this version and download it too
5. Wait until the downloads are finised (esc -> Downlads -> enter)
6. Exit links application with `q`

Now we should be able to see both files in this directory.
```
# ls
```
\
Import the gentoo release signing key.
```
# gpg --import /usr/share/openpgp-keys/gentoo-release.asc
```
\
Verify the files we just downloaded.
```
# gpg --verify ./<file>.asc
```
This should tell us that it is a good signature.

Get rid of the .asc file, because the verification is done.
```
# rm ./<file>.asc
```
\
Move the remaining file into the root directory of our system that we are installing.
```
# mv ./<file> /mnt/gentoo
```
\
Change into that root directory.
```
# cd /mnt/gentoo/
```
\
Unpack the stage3 file.
```
# tar xpvf ./<file> --xattrs-include="*.*" --numeric-owner
```
\
Delete the .tar.xz file.
```
# rm ./<file>
```
\
If we look into the root directory now we can se our familiar linux filestructure.
```
# ls
```

### 15. Adjust swap settings
Change swap options in config file to allow the swapfile to be in a local filesystem. To do this we have to uncomment these two options.
```
# nano ./etc/conf.d/swap
```

_/mnt/gentoo/etc/conf.d/swap_
```
...
rc_before="!localmount"
...
rc_need="localmount"
...
```

### 16. Set locales
First we have to adjust the locale.gen file. Add or uncomment the locales you need (in this case i am using the us ones).
You have to enter the files like follows.
```
# nano ./etc/locale.gen
```

_/mnt/gentoo/etc/locale.gen_
```
en_US ISO-8859-1
en_US.UTF-8 UTF-8
```
\
We have to adjust one more file.
```
# nano ./etc/locale.conf
```

_/mnt/gentoo/etc/locale.conf_
```
LANG="en_US.UTF-8 UTF-8"
LC_COLLATE="C.UTF-8 UTF-8"
```

### 17. Check clock
Check if the hwclock is set right.
```
# nano ./etc/conf.d/hwclock
```

_/mnt/gentoo/etc/conf.d/hwclock_
```
clock="UTC"
```

### 18. Set keymap
You can check all available keymaps here.
```
# ls /usr/share/keymaps/i386/qwertz/
```
or
```
# ls /usr/share/keymaps/i386/qwerty/
```
Just remove the .map.gz and you have the keymap name.
\
We can write that into this file.
```
# nano ./etc/conf.d/keymaps
```

_/mnt/gentoo/etc/conf.d/keymaps_
```
keymap="de-latin1"
```
\
And this file.
```
# nano ./etc/vconsole.conf
```

_/mnt/gentoo/etc/vconsole.conf_
```
KEYMAP=de-latin1
```

### 19. Set timezone
You can check all available timezones here.
```
# ls /usr/share/zoneinfo/
```
and for Europe for example
```
# ls /usr/share/zoneinfo/Europe/
```
This results in something like "Europe/Berlin"
\
Now you can write that into the timezone file.
```
# echo "Europe/Berlin" > ./etc/timezone
```

### 20. Create FS tables
First we have to figure out the UUIDs of our boot partition. (I recommend to call this command in a separate tty for ease of use (ctsl + alt + f1/f2/f3/...) of copying.)
```
# blkid /dev/<name-of-boot-partition>
```
\
Adjust the fstab file.
```
# nano ./etc/fstab
```

_/mnt/gentoo/etc/fstab_
```
# <fs>                  <mountpoint>        <type>        <opts>                                                                      <dump>    <pass>
LABEL=BTROOT            /                   btrfs         defaults,noatime,compress=lzo,autodefrag,discard=async,subvol=activeroot    0         0
LABEL=BTROOT            /home               btrfs         defaults,noatime,compress=lzo,autodefrag,discard=async,subvol=home          0         0
LABEL=BTROOT            /swap               btrfs         defaults,noatime,compress=lzo,autodefrag,discard=async,subvol=swap          0         0
/swap/swapfile          none                swap          defaults                                                                    0         0
UUID=<boot-part-uuid>   /boot               vfat          umask=077                                                                   0         1
UUID=<boot-part-uuid>   /efi                vfat          umask=077                                                                   0         1
```

### 21. Setup initramfs
To do this we have to create a configuration in _/etc/dracut.conf.d/_.
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

### 22. Setup grub
Setup the grub configuration. 
```
# nano ./etc/default/grub
```

_/mnt/gentoo/etc/default/grub_
```
GRUB_DISABLE_LINUX_PARTUUID=false
GRUB_DISTRIBUTOR="Gentoo"
GRUB_TIMEOUT=3

GRUB_ENABLE_CRYPTODISK=y
GRUB_CMDLINE_LINUX_DEFAULT=""
```

### 23. Setup portage
Copy the repos.conf into the portage folder.
```
# mkdir ./etc/portage/repos.conf
# cp ./usr/share/portage/config/repos.conf ./etc/portage/repos.conf/gentoo.conf
```
\
Download the config files from github.
```
# wget https://github.com/desolatedependency/my-minimal-portage-configs/archive/main.zip
```
\
Unzip the file.
```
# unzip ./main.zip
# rm ./main.zip
```
\
Move the files into the right directories.
```
# mv ./my-minimal-portage-configs-main/* ./etc/portage/
# rm -r ./my-minimal-portage-configs-main/
# mkdir ./etc/portage/env
# mv ./etc/portage/no-lto ./etc/portage/env/
```
> [!NOTE]
> You can overwrite everything that got autogenerated.

Adjust make.conf. This is different between different hardware.

_/mnt/gentoo/etc/portage/make.conf_
```
...
MAKEOPTS="-j16 -l16"
...
VIDEO_CARDS="amdgpu radeonsi intel"
...
```
The marked video drivers are for a system that has a intel and a amd gpu. 'intel i915' for intel hd 3000 and older. 
\
Select a mirror that is close to you. (Choose at least one https and one rsync mirror.)
```
# mirrorselect -i -o >> ./etc/portage/make.conf
```
1. Select with 'space'
2. Apply with 'enter'

Comment the added lines in make.conf.

_/mnt/gentoo/etc/portage/make.conf_
```
# Set package mirrors
GENTOO_MIRRORS= ...
```

### 24. Change into root
Copy resolve.conf for internet usage.
```
# cp /etc/resolv.conf /mnt/gentoo/etc/
```
\
Mount a few things from our live cd to our root system.
```
# mount --types proc /proc/ /mnt/gentoo/proc/
# mount --rbind /sys/ /mnt/gentoo/sys/
# mount --rbind /dev/ /mnt/gentoo/dev/
# mount --bind /run/ /mnt/gentoo/run/
# mount --make-rslave /mnt/gentoo/sys/
# mount --make-rslave /mnt/gentoo/dev/
# mount --make-slave /mnt/gentoo/run/
```
\
Chroot into root system.
```
# chroot /mnt/gentoo/ /bin/bash
# source /etc/profile
# export PS1="(chroot) ${PS1}"
```

### 25. [If interrupted]
If the installation gets interrupted from this point out, or we have to chroot back to fix something, we can continue by going through these steps.

Redo the [Preparation](#5-preparation) and [Connect to wifi (if not connected over ethernet)](#7-connect-to-wifi-if-not-connected-over-ethernet) steps.

Now we have to mount all of the drives again. To do that we unlock our encrypted drive.
```
# cryptsetup luksOpen /dev/<name-of-root-partition> cryptroot
```
\
Then we are mounting root.
```
# mkdir /mnt/root
# mount -t btrfs -o defaults,noatime,compress=lzo,autodefrag /dev/mapper/cryptroot /mnt/root/
```
\
And then we mount the other volumes.
```
# mount -t btrfs -o defaults,noatime,compress=lzo,autodefrag,subvol=activeroot /dev/mapper/cryptroot /mnt/gentoo/
# mount -t btrfs -o defaults,noatime,compress=lzo,autodefrag,subvol=swap /dev/mapper/cryptroot /mnt/gentoo/swap/
# mount -t btrfs -o defaults,noatime,compress=lzo,autodefrag,subvol=home /dev/mapper/cryptroot /mnt/gentoo/home/
# mount /dev/<name-of-boot-partition> /mnt/gentoo/boot/
# mount /dev/<name-of-boot-partition> /mnt/gentoo/efi/
```
\
Turn on swap.
```
# swapon /mnt/gentoo/swap/swapfile
```
\
After following the steps in [Change into root](#24-change-into-root) we can continue from the point where we stopped or start debugging.

### 26. Updating the Gentoo ebuild repository
Syncronize portage with mirrors.
```
# emerge-webrsync
# emerge --sync --quiet
```

### 27. Select Profile
You can list all available profiles with:
```
eselect profile list
```
\
Select a fitting profile for your setup. (In my case i am using the desktop profile)
```
# eselect profile set <number-of-profile>
```

### 28. Applying localization configurations
Sync timezone config.
```
# emerge --config sys-libs/timezone-data
```
\
Generate locales.
```
# locale-gen
```
\
Update environment variables for this terminal.
```
# env-update
```
\
Now we can see all the usable locales here.
```
# eselect locale list
```
\
Select the locale you want to use for your system. (In my case en_US.utf8)
```
# eselect locale set <number-of-locale>
```
\
Restart terminal.
```
# source /etc/profile
# export PS1="(chroot) ${PS1}"
```

### 29. Set cpu flags
Install cpuid2cpuflags.
```
# emerge --ask --oneshot app-portage/cpuid2cpuflags
```
\
If you want you can see all your cpu flags by running it.
```
# cpuid2cpuflags
```
This will output the flags that should be used for the cpu.
\
We want to copy these into our _package.use_:
```
# echo "*/* $(cpuid2cpuflags)" > /etc/portage/package.use/00cpu-flags
```

### 30. Updating the @world set
We will update the @world set to recompile everything that got its flags changed.
```
# emerge --ask --verbose --update --deep --changed-use @world
```
> [!NOTE]
> This will probably take a long time.

Now we want to check if there any obsolete packages.
```
# emerge --ask --pretend --depclean
```
If there are packeges that you want to keep use `emerge --noreplace` to add it to the @world list.
\
To delete the obsolete packages you have to call the same command as above without the --pretend.
```
# emerge --ask --depclean
```

### 31. Install core packages
Emerge all core packages.
```
# emerge --ask sys-kernel/installkernel sys-kernel/linux-firmware \
> sys-fs/cryptsetup sys-fs/btrfs-progs sys-block/parted sys-boot/grub sys-auth/seatd  sys-apps/pciutils sys-process/cronie \
> net-misc/chrony net-misc/networkmanager app-admin/sysklogd app-admin/doas \
> app-editors/neovim
```
> [!NOTE]
> This will also take a long time.

### 32. Configure doas
Create a doas.conf.
```
# nano /etc/doas.conf
```

_/etc/doas.conf_
```
permit :wheel
```
\
Adjust permissions for doas.conf.
```
# chown -c root:root /etc/doas.conf
# chmod -c 0400 /etc/doas.conf
```

### 33. Configuring services
Service setup.
```
# rc-update add seatd boot
# rc-update add dbus boot
# rc-update add NetworkManager boot
# rc-update add sysklogd default
# rc-update add chronyd default
# rc-update add cronie default
```
\
Replace the service that sets the hostname.
```
# rc-update delete hostname boot
# rc-service NetworkManager start
# nmcli general hostname <hostname-of-system>
```

### 34. Managing user accounts
Set a password for root.
```
# passwd
```
\
Setup user account.
```
# useradd <username>
# passwd <username>
# usermod <username> -aG users
# usermod <username> -aG wheel
# usermod <username> -aG disk
# usermod <username> -aG cdrom
# usermod <username> -aG floppy
# usermod <username> -aG audio
# usermod <username> -aG video
# usermod <username> -aG input
# usermod <username> -aG seat
```

### 35. Compile kernel
Now we are going to compile the kernel.
```
# emerge --ask sys-kernel/gentoo-kernel
```
> [!NOTE]
> This will also take a long time.

\
Once the kernel is build it will be updated with the package manager. Old kernels will be deleted with a depclean.
```
# emerge --ask --depclean
```
\
Alternatively, to specifically clean up old kernel versions:
```
# emerge --ask --prune sys-kernel/gentoo-kernel sys-kernel/gentoo-kernel-bin
```

### 36. Install bootloader
Install grub.
```
# grub-install --target=x86_64-efi --efi-directory /boot/ --removable --recheck
```
\
Generate grub boot option configuration.
```
# grub-mkconfig -o /boot/grub/grub.cfg
```

### 37. Rebooting
Rebooting the system.
```
# reboot
```
> [!NOTE]
> Now you can remove the installation device.

### 38. Lock root password
After logging in with our user account we are locking the root password for safty reasons.
```
~ doas -u root passwd -l root
```

### 39. Connect Wifi
You can connect over wifi with this menu.
```
~ nmtui
```
