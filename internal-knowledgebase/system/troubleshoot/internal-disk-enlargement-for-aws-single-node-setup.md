---
tool_name: cos
doc_type: troubleshoot
category: system
title: "INTERNAL - Disk enlargement for AWS single node setup"
summary: "There are two cases for disk enlargement:"
---
# INTERNAL - Disk enlargement for AWS single node setup

There are two cases for disk enlargement:

* There is a dedicated disk for storage.
* There is no dedicated storage disk

## How to enlarge disk with dedicated disk

## Step 1

Modify the disk size using AWS console

## Step 2

Check the kernel messages with dmesg to make sure that the new disk size is recognized

## Step 3

Increase disk size with parted. For example:

```
(parted) resizepart 2
End?  [108GB]? 100%
Error: Error informing the kernel about modifications to partition /dev/xvdb2 -- Invalid argument.  This means Linux won't know about any changes you made to /dev/xvdb2 until you reboot -- so you shouldn't mount
it or use it in any way before rebooting.
--------------------
Ignore/Cancel? Ignore
--------------------
(parted) print
Model: Xen Virtual Block Device (xvd)
Disk /dev/xvdb: 215GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name     Flags
 1      1049kB  8389kB  7340kB               primary  msftdata
 2      8389kB  215GB   215GB                primary  msftdata

(parted) quit
Information: You may need to update /etc/fstab.
```

## Step 4

partprobe

```
cluster1 [root@n0010 ec2-user]# partprobe
Error: Error informing the kernel about modifications to partition /dev/xvda4 -- Invalid argument.  This means Linux won't know about any changes you made to /dev/xvda4 until you reboot -- so you shouldn't mount it or use it in any way before rebooting.
Error: Partition(s) 4 on /dev/xvda have been written, but we have been unable to inform the kernel of the change, probably because it/they are in use.  As a result, the old partition(s) will remain in use.  You should reboot now before making further changes.
Error: Error informing the kernel about modifications to partition /dev/xvdb2 -- Invalid argument.  This means Linux won't know about any changes you made to /dev/xvdb2 until you reboot -- so you shouldn't mount it or use it in any way before rebooting.
Error: Partition(s) 2 on /dev/xvdb have been written, but we have been unable to inform the kernel of the change, probably because it/they are in use.  As a result, the old partition(s) will remain in use.  You should reboot now before making further changes.
```

## Step 5

Reboot the node

## Step 6

partprobe

```
cluster1 [root@n0010 ec2-user]# partprobe
Error: Error informing the kernel about modifications to partition /dev/xvda4 -- Invalid argument.  This means Linux won't know about any changes you made to /dev/xvda4 until you reboot -- so you shouldn't mount it or use it in any way before rebooting.
Error: Partition(s) 4 on /dev/xvda have been written, but we have been unable to inform the kernel of the change, probably because it/they are in use.  As a result, the old partition(s) will remain in use.  You should reboot now before making further changes.
```
## Step 7

Reboot the node (for some reason after the first reboot storage does not work, so it requires the second reboot)

## Step 8

Enlarge the disk on Exastorage

## How to enlarge disk without dedicated disk

## Step 1

Remove partitions 6 and 7

```
cluster1 [root@n0010 ~]# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/xvda2       50G  3.9G   43G   9% /
tmpfs           3.9G   48K  3.9G   1% /dev/shm
/dev/xvda1      243M   42M  189M  19% /boot
/dev/xvda5       25G  3.7G   20G  16% /usr/opt
cluster1 [root@n0010 ~]# parted
GNU Parted 3.2
Using /dev/xvda
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) print
Model: Xen Virtual Block Device (xvd)
Disk /dev/xvda: 215GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:

Number  Start   End     Size    Type      File system     Flags
 1      1049kB  263MB   262MB   primary   ext2            boot
 2      263MB   54.0GB  53.7GB  primary   ext4
 3      54.0GB  62.5GB  8590MB  primary   linux-swap(v1)
 4      62.5GB  215GB   152GB   extended
 5      62.5GB  89.4GB  26.8GB  logical   ext4
 6      89.4GB  89.4GB  8389kB  logical
 7      89.4GB  215GB   125GB   logical

(parted) rm 7
Error: Error informing the kernel about modifications to partition /dev/xvda4 -- Invalid argument.  This means Linux won't know about any changes you made to /dev/xvda4 until you reboot -- so you shouldn't mount
it or use it in any way before rebooting.
--------------------
Ignore/Cancel? Ignore
--------------------
(parted) print
Model: Xen Virtual Block Device (xvd)
Disk /dev/xvda: 215GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:

Number  Start   End     Size    Type      File system     Flags
 1      1049kB  263MB   262MB   primary   ext2            boot
 2      263MB   54.0GB  53.7GB  primary   ext4
 3      54.0GB  62.5GB  8590MB  primary   linux-swap(v1)
 4      62.5GB  215GB   152GB   extended
 5      62.5GB  89.4GB  26.8GB  logical   ext4
 6      89.4GB  89.4GB  8389kB  logical

(parted) rm 6
Error: Error informing the kernel about modifications to partition /dev/xvda4 -- Invalid argument.  This means Linux won't know about any changes you made to /dev/xvda4 until you reboot -- so you shouldn't mount
it or use it in any way before rebooting.
--------------------
Ignore/Cancel? Ignore
--------------------
(parted) print
Model: Xen Virtual Block Device (xvd)
Disk /dev/xvda: 215GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:

Number  Start   End     Size    Type      File system     Flags
 1      1049kB  263MB   262MB   primary   ext2            boot
 2      263MB   54.0GB  53.7GB  primary   ext4
 3      54.0GB  62.5GB  8590MB  primary   linux-swap(v1)
 4      62.5GB  215GB   152GB   extended
 5      62.5GB  89.4GB  26.8GB  logical   ext4
```

## Step 2

Resize partition 5

```
(parted) resizepart
Partition number? 5
Warning: Partition /dev/xvda5 is being used. Are you sure you want to continue?
Yes/No? Yes
End?  [89.4GB]? 100%
Error: Error informing the kernel about modifications to partition /dev/xvda4 -- Invalid argument.  This means Linux won't know about any changes you made to /dev/xvda4 until you reboot -- so you shouldn't mount
it or use it in any way before rebooting.
--------------------
Ignore/Cancel? Ignore
--------------------
Error: Error informing the kernel about modifications to partition /dev/xvda5 -- Invalid argument.  This means Linux won't know about any changes you made to /dev/xvda5 until you reboot -- so you shouldn't mount
it or use it in any way before rebooting.
--------------------
Ignore/Cancel? Ignore
--------------------
(parted) print
Model: Xen Virtual Block Device (xvd)
Disk /dev/xvda: 215GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:

Number  Start   End     Size    Type      File system     Flags
 1      1049kB  263MB   262MB   primary   ext2            boot
 2      263MB   54.0GB  53.7GB  primary   ext4
 3      54.0GB  62.5GB  8590MB  primary   linux-swap(v1)
 4      62.5GB  215GB   152GB   extended
 5      62.5GB  215GB   152GB   logical   ext4

(parted) quit
Information: You may need to update /etc/fstab.
```

## Step 3

partprobe and reboot

```
cluster1 [root@n0010 ~]# partprobe
Error: Error informing the kernel about modifications to partition /dev/xvda4 -- Invalid argument.  This means Linux won't know about any changes you made to /dev/xvda4 until you reboot -- so you shouldn't mount it or use it in any way before rebooting.
Error: Error informing the kernel about modifications to partition /dev/xvda5 -- Invalid argument.  This means Linux won't know about any changes you made to /dev/xvda5 until you reboot -- so you shouldn't mount it or use it in any way before rebooting.
Error: Partition(s) 4, 5 on /dev/xvda have been written, but we have been unable to inform the kernel of the change, probably because it/they are in use.  As a result, the old partition(s) will remain in use.  You should reboot now before making further changes.
cluster1 [root@n0010 ~]# reboot -h now
```

## Step 4

resize the filesystem

```
cluster1 [root@n0010 ~]# resize2fs /dev/xvda5
resize2fs 1.41.12 (17-May-2010)
Filesystem at /dev/xvda5 is mounted on /usr/opt; on-line resizing required
old desc_blocks = 9, new_desc_blocks = 28
Performing an on-line resize of /dev/xvda5 to 115803136 (4k) blocks.
The filesystem on /dev/xvda5 is now 115803136 blocks long.
```

## Step 5

Stop the Exastorage

## Step 6

Increase the file size for EXAStorage disk using truncate (Don't use dd, it will crash the system)

```
cluster1 [root@n0010 ~]# ls -lh /usr/opt/exastorage_disk2
-rw-r--r-- 1 root root 37G Sep 27 14:59 /usr/opt/exastorage_disk2
cluster1 [root@n0010 ~]# truncate --size=+300GB /usr/opt/exastorage_disk2
cluster1 [root@n0010 ~]# ls -lh /usr/opt/exastorage_disk2
-rw-r--r-- 1 root root 317G Sep 27 15:23 /usr/opt/exastorage_disk2
```

## Step 7

If you see that EXAoperation does not respond restart the cos

## Step 8

Start the Exastorage

## Step 9

Enlarge the device using cshdd command

```
cluster1 [root@n0010 ~]# cshdd --enlarge -n 10 -h /dev/disk_d03_storage_0
Successfully enlarged HDD /dev/disk_d03_storage_0 on node 10.
```
