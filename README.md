# nicOS
###### A LUKS encrypted linux distro for software developers. 



&nbsp;
## Installation:
#### Step one - Update System Clock
###### make sure your clock is accurate
```sh
timedatectl set-ntp true
timedatectl status
```
***



&nbsp;
#### Step two - Setup Partitions
###### load device mapper and encryption kernel modules
```sh
modprobe -a dm-mod dm_crypt
```
###### find the name of your disk (e.g., /dev/sda)
```sh
fdisk -l
```
###### erase the partition tables on your disk
```sh
sgdisk --zap-all /dev/sda
```
***



&nbsp;
#### Step three - Create Partitions
###### creating new GPT entries
```sh
gdisk /dev/sda
```
###### use the 'o' option to create a new GPT.
```sh
Command (? for help): o 
This option deletes all partitions and creates a new protective MBR.
Proceed? (Y/N): Y
```
###### use the 'n' option to create a new partition
```sh
Command (? for help): n
```
###### press 'Enter' to use the first partition
```sh
Partition number (1-128, default 1):
```
###### press 'Enter' to select the default option for the first sector of the partition
```sh
First sector (34-31457246, default = 2048) or {+-}size{KMGTP}:
````
###### enter '+1M' for the last sector of the partition
```sh
Last sector (2048-31457246, default = 31457246) or {+-}size{KMGTP}: +1M
```
###### enter 'ef02' as the partition type
```sh
Current type is 'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300): ef02
```
###### use the 'n' option to create a new partition
```sh
Command (? for help): n
```
###### then press 'Enter' to use the second partition
```sh
Partition number (2-128, default 2):
```
###### press 'Enter' to select the default option for the first sector of the partition
```sh
First sector (34-31457246, default = 4096) or {+-}size{KMGTP}:
```
###### then '+512M' for the last.
```sh
Last sector (4096-31457246, default = 31457246) or {+-}size{KMGTP}: +512M
```
###### press 'Enter' to select '8300'
```sh
Current type is 'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300):
```
###### use the 'n' option to create a new partition
```sh
Command (? for help): n
```
###### then press 'Enter' to use the third partition.
```sh
Partition number (3-128, default 3):
```
###### press 'Enter' to select the default option for the first sector of the partition
```sh
First sector (34-31457246, default = 1052672) or {+-}size{KMGTP}:
```
###### then '+0' to take up the remainder of the free space
```sh
Last sector (1052672-31457246, default = 31457246) or {+-}size{KMGTP}: +0
```
###### press 'Enter' to select '8300'
```sh
Current type is 'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300):
```
###### use the 'p' option to preview your changes.
```sh
| Number | Start       | End      | Size       | Code | Name                |
| 1      | 2048        | 4095     | 1024.0 Kib | EF02 | BIOS boot partition |
| 2      | 4096        | 1052671  | 512.0 Mib  | 8300 | Linux filesystem    |
| 3      | 1052672     | 67108830 | 31.5 Gib   | 8300 | Linux filesystem    |
```

###### use the 'w' option to write your changes to disk then 'y' to proceed
```sh
Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!
Do you want to proceed? (Y/N): y
```
###### reboot the kernel
```sh
# reboot
```
***



&nbsp;
#### Step four - Setup encryption
###### setup encryption on the partition (/dev/sda3)
```sh
cryptsetup -v -y -c aes-xts-plain64 -s 512 -h sha512 -i 5000 --use-urandom luksFormat /dev/sda3
```
###### type 'YES' to begin encryption
```sh
WARNING!!!
==========
This will overwrite data on /dev/sda3 irrevocably.
Are you sure? (Type UPPERCASE YES): YES
```
***



&nbsp;
#### Step five - Create filesystems
###### unlock the LUKS device to setup filesystems
```sh
cryptsetup open /dev/sda3 cryptroot
```
###### create filesystems for '/boot' and '/'
```sh
mkfs.ext4 /dev/sda2
mkfs.ext4 /dev/mapper/cryptroot
```
###### mount the filesystems
```sh
mount /dev/mapper/cryptroot /mnt
mkdir /mnt/boot
mount /dev/sda2 /mnt/boot
```
###### display the filesystems using the 'lsblk' command
```sh
lsblk /dev/sda
```
***



&nbsp;
#### Step six – Select a mirror
###### use vi to edit the mirrorlist
```sh
vi /etc/pacman.d/mirrorlist
```
###### refresh the package lists
```sh
pacman -Syy
```
***



&nbsp;
#### Step seven – Install the base system
###### use the pacstrap command to install the system to /mnt
```sh
pacstrap -i /mnt base base-devel
```
***



&nbsp;
#### Step eighth – Generate a fstab
###### use the command below to generate one the fstab file
```sh
genfstab -U /mnt >> /mnt/etc/fstab
```
###### to check the fstab file for errors.
```sh
blkid
```
###### cat out the /mnt/etc/fstab file and compare the UUIDs and logical volumes types to what was returned by blkid above.
```sh
cat /mnt/etc/fstab
```
