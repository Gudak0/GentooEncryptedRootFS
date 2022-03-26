# Gentoo Encrypted Root FS
Written in an MD file on github since it looks nice. any additional files will also be linked here

## Step 1: Prepping disks
Here is how I partition my disks

| Partition Name  | Format  | Size  | Type            |
|-----------------|---------|-------|---              |
| Boot            | fat32   | 256MB | EFI             |
| Root            | LUIKS   | Rest of Disk  | RootFS  |

