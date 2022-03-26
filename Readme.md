# Gentoo Encrypted Root FS
Written in an MD file on github since it looks nice. any additional files will also be linked here

## Step 1: Prepping disks
Here is how I partition my disks

| Partition Name  | Format  | Size  | Type            |
|-----------------|---------|-------|---              |
| Boot            | FAT32   | 256MB | EFI             |
| Root            | LUKS   | Rest of Disk  | RootFS  |

To partiton it we can do this
```
parted /dev/disklabel
```
in this example, and for the rest of the guide, the disk will be /dev/sda but it may be something like /dev/nvme0n1 for NVMe devices for example.

So I would run
```
parted /dev/sda
```
Note with parted: make sure you point to the disk device and NOT a partiton, very important

and here is the rest of the partitioning goes
```
(parted) unit mb
```
This tells Parted to write using megabytes
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
and check if the changes were made correctly by doing
```
fdisk -l
```
or
``` lsblk ```
To see if they were written correctly, if so, continue to Step 2 below
