---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "Resizing /d02_data disk device in AWS"
summary: "Expand the `/d02_data` (`/spool`) partition on AWS data nodes when spool capacity becomes insufficient."
---
# Resizing /d02_data disk device in AWS

## Purpose

Increase `/spool` capacity on AWS-hosted clusters by extending the underlying EBS volume and filesystem.

## Impact

- Requires maintenance window.
- Database and storage services must be stopped and affected nodes rebooted.

## Prerequisites

- `/spool` partition is the last partition on disk.
- Access to AWS console for EBS changes.
- Access to node shell as root.
- EXAoperation access (for legacy deployments where disk size metadata is managed there).

## Procedure

1. In EXAoperation (legacy clusters), update node disk configuration:
- `Nodes -> Select affected nodes -> Install flag`
- `Nodes -> Disks -> d02_data -> Edit -> clear default fixed size (45 GiB)`
- `Nodes -> Select affected nodes -> Active flag`

2. In AWS, expand the corresponding EBS volumes for all affected instances.

3. Wait until EBS volume optimization reaches `100%`.

4. On each affected node, verify kernel detects new capacity:

```bash
dmesg | grep "detected capacity"
```

Example:

```powershell
cluster1 [root@n0011 ~]# dmesg | grep "detected capacity"
Nov 2 16:29:10 n0011 kernel: [23590.335884] nvme1n1: detected capacity change from 107374182400 to 214748364800
```

5. Extend the `/spool` partition with `parted` on each affected node:

```powershell
cluster1 [root@n0011 ~]# parted /dev/nvme1n1
GNU Parted 3.2
Using /dev/nvme1n1
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) print
*** Respond with "Fix" if a Warning is received ***
*** Partition #4 should be your /spool partition. Double-check the size ***
(parted) resizepart 4
Warning: Partition /dev/nvme1n1p4 is being used. Are you sure you want to continue?
Yes/No? Yes
End? [101GB]? 100%
(parted) print
*** Check if partition #4 has been extended ***
(parted) quit
```

6. Stop database and storage services, then reboot all affected nodes.

7. After reboot, grow the filesystem on each node:

```powershell
cluster1 [root@n0011 ~]# resize2fs /dev/nvme1n1p4
resize2fs 1.42.9 (28-Dec-2013)
Filesystem at /dev/nvme1n1p4 is mounted on /d02_data; on-line resizing required
old_desc_blocks = 6, new_desc_blocks = 19
The filesystem on /dev/nvme1n1p4 is now 39581682 blocks long.
```

8. Validate new size:

```bash
df -h /spool
```

9. In EXAoperation, verify updated `d02_data` disk size is reflected.

## Notes

- Always confirm the target partition number before `resizepart`.
- For newer cloud-native workflows, also check infrastructure scaling options in:
  - `documents/cos/confd-system-and-infrastructure.md` (`infra_db_scale` and related operations)
