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
