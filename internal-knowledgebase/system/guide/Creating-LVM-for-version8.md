---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "Create LVM layout for Exasol v8"
summary: "Prepare LVM volumes on LUKS-encrypted disks for Exasol v8 storage deployment."
---
# Create LVM layout for Exasol v8

## Purpose

Prepare storage disks with LVM on top of LUKS devices before Exasol v8 setup.

## Example layout

- `/dev/sda`: OS disk.
- `/dev/sdb` -> `/dev/mapper/exa1`: encrypted storage disk.
- `/dev/sdc` -> `/dev/mapper/exa2`: encrypted storage disk.

## Prerequisites

- OS installed and bootable.
- LUKS already configured and unlocked for target storage disks.
- Sudo/root access.

## Procedure

### 1. Verify disks

```shell
lsblk
```

Confirm you are not using OS/root partition for storage PV creation.

### 2. Create physical volumes (PV)

```shell
pvcreate /dev/mapper/exa1 /dev/mapper/exa2
pvs
```

### 3. Create volume groups (VG)

```shell
vgcreate exa1 /dev/mapper/exa1
vgcreate exa2 /dev/mapper/exa2
vgs
```

### 4. Create logical volumes (LV)

Use most of available capacity while keeping buffer:

```shell
lvcreate -l 95%FREE exa1
lvcreate -l 95%FREE exa2
lvs
```

Typical result: `exa1/lvol0` and `exa2/lvol0`.

### 5. Final verification

```shell
lsblk
```

Expected: encrypted mapper devices with LVM logical volumes beneath each storage disk.

## Notes

- Keep naming consistent (`exa1`, `exa2`, `lvol0`, etc.) to match deployment expectations.
- Reserve free space buffer (for example 5%) for operational flexibility.


