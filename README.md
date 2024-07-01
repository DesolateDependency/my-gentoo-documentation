# My-Gentoo
## Gentoo installation
I used the following sources to collect information:
- Gentoo amd64 handbook (https://wiki.gentoo.org/wiki/Handbook:AMD64/Full/Installation)
- Arch wiki (https://wiki.archlinux.org/title/Btrfs)
- Youtube video by libreisaac (https://youtu.be/t0nPxDlFL2I?si=FyU1lBa7gS3blI2J)
- My dying braincell

### Preparation
After booting into the minimal install on the boot device it promts for a keyboard layout. If chosen "de" a few of the keys are still wrong. So i set the keymap manualy.
```
# loadkeys de-latin1
```

### Testing the network
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

### Connect to wifi
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

### Sync network time
We will run chronyd to sync up our time.
```
# chronyd -q
```

### Partitioning
To check what drives are currently connected we can call this command.
```
# lsblk
```
\
At the beginning we want to override all data that was on it bevor for safety reasons.
```
# dd if=/dev/urandom of=/dev/<name-of-drive> bs=4096 status=progress
```
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

### Encrypting root
First we have to encrypt our root and open it.
```
# cryptsetup luksFormat -s256 -c aes-xts-plain64 /dev/<name-of-root-partition>
# cryptsetup luksOpen /dev/<name-of-root-partition> cryptroot
```

### Formating & mounting partitions
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

### Create and mount subvolumes
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
# mount /dev/sda1 /mnt/gentoo/boot/
# mount /dev/sda1 /mnt/gentoo/efi/
```

### Create swap
Create swap.
```
# btrfs filesystem mkswapfile --size 16g --uuid clear /mnt/gentoo/swap/swapfile
```
\
Activate the swap file.
```
# swapon /mnt/gentoo/swap/swapfile
```

### Downloading, varifying and unpacking stage3
Enter gentoos text based download page.
```
# links https://distfiles.gentoo.org/releases/amd64/autobuilds
```
1. Navigate to the latest timestamp directory.
2. Go to the "stage3-amd64-hardened-openrc-..." file
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

### Adjust swap settings
Change swap options in config file to allow the swapfile to be in a local filesystem. To do this we have to uncomment these two options.
```
# nano ./etc/conf.d/swap
```

_/mnt/gentoo/etc/conf.d/swap_
```
...
rc_before="!localmount"
...
rc_need="netmount"
...
```

### Set locales
First we have to adjust the locale.gen file. Add or uncomment the locales you need (in this case i am using the us ones).
Youhave to enter the files like follows.
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

### Check clock
Check if the hwclock is set right.
```
# nano ./etc/conf.d/hwclock
```

_/mnt/gentoo/etc/conf.d/hwclock_
```
clock="UTC"
```

### Set keymap
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

### Set timezone
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

### Create FS tables
First we have to figure out the UUIDs of our boot partition.
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

### Setup grub
For this we need the UUID from our root partition.
```
# blkid /dev/<name-of-root-partition>
```
\
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
GRUB_CMDLINE_LINUX_DEFAULT="crypt_root=UUID=<root-partition-uuid> quiet"
```

### Setup portage
Copy the repos.conf into the portage folder.
```
# mkdir ./etc/portage/repos.conf
# cp ./usr/share/portage/config/repos.conf ./etc/portage/repos.conf/gentoo.conf
```
\
Download the config files from github.
```
# wget https://github.com/desolatedependency/my-portage-configs/archive/main.zip
```
\
Unzip the file.
```
# unzip ./main.zip
# rm ./main.zip
```
\
Remove premade portage configs.
```
# rm -rf ./etc/portage/package.accept_keywords
# rm -rf ./etc/portage/package.use
```
\
Move the files into the right directories.
```
# mv ./my-portage-configs-main/* ./etc/portage/
# rm -r ./my-portage-configs/
# mkdir ./etc/portage/env
# mv ./etc/portage/no-lto ./etc/portage/env/
```
> [!NOTE]
> You can overwrite everything that got autogenerated.

Adjust make.conf. This is different between different hardware.

_/mnt/gentoo/etc/portage/make.conf_
```
...
MAKEOPTS="-j8 -l8"
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

### Change into root
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

### Applying localization configurations
Syncronize portage with mirrors.
```
# emerge-webrsync
# emerge --sync --quiet
```
\
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
Restart terminal.
```
# source /etc/profile
# export PS1="(chroot) ${PS1}"
```

### Set cpu flags
Install cpuid2cpuflags.
```
# emerge app-portage/cpuid2cpuflags
```
\
Run cpuid2cpuflags.
```
# cpuid2cpuflags
```
This will output the flags that should be used for the cpu.

Now we can add those flags to our make.conf file like this. It is recommended to do this in a separate tty for ease of use (alt + f1/f2/f3/...).
```
# nano /etc/portage/make.conf
```

_/etc/portage/make.conf_
```
...
# Compilation flags
CPU_FLAGS_X86="<cpu-flags>"
...
```

### Recompiling Everything
Recompile every application in our stage3 to compile it with our own flags/optimizations instead of using the precompiled ones.
```
# emerge --emptytree -a -1 @installed
```
> [!NOTE]
> This will probably take a long time.

### Compile Rust language compiler
Emerge Rust.
```
# emerge dev-lang/rust --ask
```
> [!NOTE]
> This will also take a long time.

Adjust config for Rust.
```
# nano /etc/portage/package.use
```

_/etc/portage/package.use_
```
...
dev-lang/rust nightly rust-analyzer rust-src rustfmt system-bootstrap
...
```

### Install core packages
Emerge all core packages.
```
# emerge --ask sys-kernel/gentoo-sources sys-kernel/genkernel sys-kernel/installkernel sys-kernel/linux-firmware \
> sys-fs/cryptsetup sys-fs/btrfs-progs sys-block/parted sys-boot/grub sys-apps/sysvinit sys-auth/seatd sys-apps/dbus sys-apps/pciutils sys-process/cronie \
> net-misc/chrony net-misc/networkmanager app-admin/sysklogd app-admin/doas app-shells/bash-completion dev-vcs/git \
> app-editors/neovim
```
> [!NOTE]
> This will also take a long time.

### Configure doas
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

### Configuring services
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

### Managing user accounts
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

### Compile Kernel
Tell portage that we are compiling the kernel ourself.
```
eselect kernel set 1
```
\
Start setup.
```
genkernel --luks --btrfs --keymap --no-splash --oldconfig --save-config --menuconfig --install all
```
Make the following changes in the menu. (change setting = space, enter section = enter, leave section = esc, move = arrows) \
_File systems ---> <*> Btrfs filesystem support_ \
_Cryptographic API ---> Length-preserving ciphers and modes ---> <*> (XOR Encrypt XOR with ciphertext stealing)_ \
_Gentoo Linux ---> Support for init systems, system and service managers ---> [ ]_
\
Start compiling the kernel by exiting the menu with esc and accept the save configuration prompt with enter.

### Install bootloader
Install and setup grub.
```
# grub-install --target=x86_64-efi --efi-directory /boot/ --removable --recheck
# grub-mkconfig -o /boot/grub/grub.cfg
```

### Rebooting
Rebooting the system.
```
# reboot
```
> [!NOTE]
> Now you can remove the installation device.

### Lock root password
After logging in with our user account we are locking the root password for safty reasons.
```
~ doas -u root passwd -l root
```

### Connect Wifi
You can connect over wifi with this menu.
```
~ mntui
```
