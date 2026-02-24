---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Managing and Replacing Faulty RAID Devices"
summary: "A faulty disk in a software RAID array can compromise the redundancy and performance of the system. It's important to identify and replace faulty disks promptly to maintain system..."
---
# Managing and Replacing Faulty RAID Devices

## Problem

A faulty disk in a software RAID array can compromise the redundancy and performance of the system. It's important to identify and replace faulty disks promptly to maintain system integrity. Customers may notice degraded performance or receive alerts from monitoring tools indicating RAID issues.

Check the status of the RAID arrays using:

```bash
cat /proc/mdstat
```

You can also inspect the details of each RAID device:

```bash
mdadm --detail /dev/md0
mdadm --detail /dev/md1
mdadm --detail /dev/md2
```

## Procedure

Follow these steps to manage and replace faulty RAID devices using the `mdadm` utility:

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

### . Add the New Drive

After the faulty drive has been physically replaced, add the new drive back to the RAID arrays:

```bash
mdadm --manage /dev/md0 --add /dev/sdX
mdadm --manage /dev/md1 --add /dev/sdX
mdadm --manage /dev/md2 --add /dev/sdX
```

The array should automatically begin rebuilding. You can monitor the progress using:

```bash
cat /proc/mdstat
```

Ensure that the rebuild completes successfully to restore full redundancy and performance.
