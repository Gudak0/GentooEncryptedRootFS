# Gentoo Encrypted Root FS
Written in an MD file on github since it looks nice. any additional files will also be linked here

## Step 1: Prepping disks
Here is how I partition my disks

| Partition Name  | Format  | Size  | Type            |
|-----------------|---------|-------|---              |
| Boot            | FAT32   | 256MB | EFI             |
| Root            | LUKS   | Rest of Disk  | RootFS  |

To partiton it we can do this
### Step 1a: parted
```
$ parted /dev/disklabel
```
in this example, and for the rest of the guide, the disk will be /dev/sda but it may be something like /dev/nvme0n1 for NVMe devices for example.

So I would run
```
$ parted /dev/sda
```
Note with parted: make sure you point to the disk device and NOT a partiton, very important

and here is the rest of the partitioning goes
```
(parted) unit mib
```
This tells Parted to write using mebibyte
```
(parted) mklabel gpt
```
This writes the disk in GPT format (since this is written for UEFI
```
(parted) mkpart primary 1 257
(parted) name 1 boot
(parted) mkpart primary 257 -1
(parted) name 2 root
```
These create the partitions and name them accordingly

To make sure our changes worked, run
```
(parted) print
```
To show our disk tabel, and if it looks write, do
```
(parted) quit
```
### Step 1b: cfdisk
To create a disk using cfdisk, run
```
$ cfdisk /dev/sda
```
If it asks something related to the partition type, BE SURE TO SELECT GPT

Now for partitioning

1. Click New, and type 256M
2. for /dev/sda1 thats 256M, go to Type, and set it to EFI System
3. Allocate the rest of the disk to the root fs (/dev/sda2)
4. Set the type for it to Linux Filesystem
5. Select Write, and type Yes
6. Click quit

### Step 1c: Checking if our partitioning was correct
To check if the changes were made correctly by doing
```
$ fdisk -l
```
or
``` 
$ lsblk
```
To see if they were written correctly, if so, continue to Step 2 below

## Step 2: Writing Filesystems
First off, make sure the dm-crypt module is loaded by running
``` 
$ moprobe dm-crypt
```

Create the filesystem for your EFI partition
```
$ mkfs.fat -F 32 /dev/sda1
```
For the root partition
```
$ cryptsetup luksFormat /dev/sda2
```
After this you'll want to enter the passphrase, remember it otherwise good luck getting any data off of it
Now open the encrypted drive with
```
$ cryptsetup luksOpen /dev/sda2 lvm
```

Create the LVM mapper
```
$ lvm pvcreate /dev/mapper/lvm 
```

And now to create our encrypted root partition
```
$ vgcreate vg0 /dev/mapper/lvm
$ lvcreate -l 100%FREE -n root vg0
```
You could make /var and /home seperate but I'm going full rawdog and making it all the root partition

and now to format it as EXT4
```
$ mkfs.ext4 /dev/mapper/vg0-root
```

## Step 3: Gentoo installation
Make sure ```/mnt/gentoo``` exists. If you're using the official gentooo instllation media, it should exist, else run
```
$ mkdir /mnt/gentoo
```

Now mount the partition
```
$ mount /dev/mapper/vg0-root /mnt/gentoo
```

Get the Stage 3 tarball of your choice and extract it to /mnt/gentoo using
```
$ tar xpvf stage3*.tar.xz --xattrs-include='*.*' --numeric-owner
```
From here, copy DNS information
```
$ cp --dereference /etc/resolv.conf /mnt/gentoo/etc
```
While the /mnt/gentoo directory is the active directory, run
```
$ mount -t proc /proc proc/
$ mount --rbind /dev dev/
$ mount --rbind /sys sys/
$ mount --rbind /run run/
```
For the ``/dev``, ``/sys``, and ``/run`` partitions, if you run systemd, make sure to also do 
```
$ mount --make-rslave /mnt/gentoo/dev
$ mount --make-rslave /mnt/gentoo/sys
$ mount --make-rslave /mnt/gentoo/run
``` 
for those respective directories, else continue on to the next step

Now mount the SHM partitions
```
$ test -L /dev/shm && rm /dev/shm && mkdir /dev/shm 
$ mount -t tmpfs -o nosuid,nodev,noexec shm /dev/shm 
$ chmod 1777 /dev/shm
```

Chroot into the new installation using
```
$ chroot /mnt/gentoo /bin/bash
```

From this point on, any command that starts with ```(HOST)``` means you need to run the cmd on the host, otherwise everything else is ran in the chroot

now run
```
$ source /etc/profile
```

From here on out, follow this part of the [AMD64 Handbook](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Base)

!! IMPORTANT !!
Make sure to configure /etc/portage/make.conf
!! IMPORTANT !!

## Step 4: fstab
Here is how ``/etc/fstab`` is configured
```
/dev/sda1             /boot vfat  noatime,defaults  0 1
/dev/mapper/vg0-root  /     ext4  defaults          0 1
```
If you have UUIDs for them (which you should be using) use
```
UUID=insert UUID
```
instead of something like ``/dev/sda1``

## Step 5: kernel
Install the kernel sources and related utils you'll need
```
$ emerge --ask gentoo-sources pciutils genkernel cryptsetup
```
Be sure to accept the license for ``linux-firmware``!

Make sure to symlink the kernel
```
$ eselect kernel list
$ eselect kernel set X
```

and then

```
$ genkernel --luks --lvm --no-zfs all
```
