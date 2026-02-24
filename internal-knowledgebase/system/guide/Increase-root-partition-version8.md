---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "Increase root partition (LVM) in v8"
summary: "Expand root filesystem capacity on LVM-based Exasol v8 hosts by growing partition, LV, and filesystem online."
---
# Increase root partition (LVM) in v8

## Purpose

Expand root filesystem space on an LVM-based host used by Exasol v8.

## Prerequisites

- Root access.
- Verified backup/snapshot before disk changes.
- Confirm target disk/partition mapping.

## Procedure

### 1. Inspect current layout

```shell
lsblk
```

Identify:

- Root disk and partition (example: `/dev/sda3`).
- Root logical volume (example: `/dev/ubuntu-vg/ubuntu-lv`).

### 2. Expand partition and LVM logical volume

Example:

```shell
growpart /dev/sda 3
lvextend -l +50%FREE /dev/ubuntu-vg/ubuntu-lv
```

Adjust device names and growth target for your host.

### 3. Grow filesystem online

Example for ext filesystem:

```shell
resize2fs -p /dev/mapper/ubuntu--vg-ubuntu--lv
```

## Validation

```shell
lsblk
df -h /
```

Confirm root filesystem reflects new size.

## Notes

- Filesystem command differs by FS type (for example `xfs_growfs` for XFS).
- Always validate target block device carefully to avoid destructive mistakes.


