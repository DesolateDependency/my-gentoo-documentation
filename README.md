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
Now we have to mount our partitions.
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
\
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

_/mnt/gentoo/etc/locale.gconf_
```
LANG="en_US.UTF-8 UTF-8"
LC_COLLATE="C.UTF- 8 UTF-8"
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
UUID=<boot-part-uuid>   /boot               vfat          umask=007                                                                   0         1
UUID=<boot-part-uuid>   /efi                vfat          umask=007                                                                   0         1
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
First remove all of the autogenerated config files.
```
# rm -rf ./etc/portage/packge.use/
# rm -rf ./etc/portage/packge.license/
# rm -rf ./etc/portage/packge.accept_keywords/
```
\
Copy the repos.conf into the portage folder.
```
# mkdir ./etc/portage/repos.conf
# cp ./usr/share/portage/config/repos.conf ./etc/portage/repos.conf/gentoo.conf
```
\
Download the config files from github.
```
# wget https://github.com/desolatedependency/my-gentoo/archive/main.zip
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

```
