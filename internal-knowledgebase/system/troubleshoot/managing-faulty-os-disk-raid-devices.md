---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Managing Faulty OS disk RAID Device"
summary: "A faulty disk in a software RAID array can compromise the redundancy, the performance of the system, and the server's ability to boot. It's important to identify and replace..."
---
# Managing Faulty OS disk RAID Device

## Problem

A faulty disk in a software RAID array can compromise the redundancy, the performance of the system, and the server's ability to boot. It's important to identify and replace faulty disks promptly to maintain system integrity. Customers may notice degraded performance or receive alerts from monitoring tools indicating RAID issues.
This procedure was performed for Webtrekk in dwh-exasol-prod-02-n0012 as documented in Salesforce, case 00090820 (sdd was the healthy disk and sdg was the faulty disk that was replaced).

Check the status of the RAID arrays using:

```bash
cat /proc/mdstat
```

You can also inspect the details of each RAID device:

```bash
sudo mdadm --detail /dev/md0
sudo mdadm --detail /dev/md1
sudo mdadm --detail /dev/md2
```

## Procedure

Follow these steps to manage and replace faulty OS-disk-related RAID devices using the `mdadm` utility (steps 1-2 can be skipped if the disk already failed and the customer has already replaced it):

### . Set the Faulty Drive

Mark the drive as faulty by running:

```bash
mdadm --manage /dev/md0 --fail /dev/sdX
mdadm --manage /dev/md1 --fail /dev/sdX
mdadm --manage /dev/md2 --fail /dev/sdX
```

Replace `/dev/sdX` with the identifier of the faulty drive.

### . Remove the Faulty Drive

Detach the faulty drive from the RAID arrays:

```bash
mdadm --manage /dev/md0 --remove /dev/sdX
mdadm --manage /dev/md1 --remove /dev/sdX
mdadm --manage /dev/md2 --remove /dev/sdX
```

### . Verify the partitions that the Drive should have

After the faulty drive has been physically replaced, check which partitions are present in the healthy drive (in this case sdd):

```bash
sudo lsblk
Example:
sudo lsblk
...
sdd                         8:48   0 223.6G  0 disk
├─sdd1                      8:49   0     1G  0 part  /boot/efi
└─sdd2                      8:50   0 222.5G  0 part
  └─md0                     9:0    0 222.4G  0 raid1
    └─ubuntu--vg-root--lv 253:0    0 222.4G  0 lvm   /
...
sdg                         8:96   0 223.6G  0 disk
```

### . Copy the partitions table to the new Drive

Copy the partition table from the current healthy OS-disk-related RAID device (sdd is the healthy drive and sdg is the replaced drive):

```bash
sudo sfdisk --wipe=always /dev/sdg < <(sudo sfdisk -d /dev/sdd)
```

Verify the new partitions with lsblk:

```bash
sudo lsblk
...
sdd                         8:48   0 223.6G  0 disk
├─sdd1                      8:49   0     1G  0 part  /boot/efi
└─sdd2                      8:50   0 222.5G  0 part
  └─md0                     9:0    0 222.4G  0 raid1
    └─ubuntu--vg-root--lv 253:0    0 222.4G  0 lvm   /
...
sdg                         8:96   0 223.6G  0 disk
├─sdg1                      8:97   0     1G  0 part
└─sdg2                      8:98   0 222.5G  0 part
```

### . Copy the data of the boot partition

Copy the data from the boot partition in the current healthy OS-disk-related RAID device to the new boot partition in the new drive (sdd1 to sdg1 in this case):

```bash
sudo dd if=/dev/sdd1 of=/dev/sdg1 bs=64K status=progress conv=noerror,sync
```

### . Add the needed OS-root partition to the raid device md0

```bash
mdadm /dev/md0 --add /dev/sdg2
```

The array should automatically begin rebuilding. You can monitor the progress using:

```bash
cat /proc/mdstat
OR
sudo mdadm --detail /dev/md0
```

Ensure that the rebuild completes successfully to restore full redundancy and performance.
