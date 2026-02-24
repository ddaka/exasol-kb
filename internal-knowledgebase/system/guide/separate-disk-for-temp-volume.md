---
tool_name: confd_client
doc_type: guide
category: system
title: "Separate Disk for Temporary Volume and Persistent Volume in a Database"
summary: "This procedure walks through adding **temporary storage disk** to Exasol cluster nodes and configuring the database to use them via `-temp_hdd_type=temporary`. This is typically..."
---
# Separate Disk for Temporary Volume and Persistent Volume in a Database

## Overview

This procedure walks through adding **temporary storage disk** to Exasol cluster nodes and configuring the database to use them via `-temp_hdd_type=temporary`. This is typically done when you want to separate the storage disk for persistent and temporary volumes.

## Prerequisites

- Approved **maintenance window** and change ticket
- **Full cluster backup** plan
- Console/iDRAC access to each server
- Admin access to the cluster CLI (tools: `c4`, `confd_client`, `csinfo`)

## How to add storage disk and enable `temp_hdd_type` for the temporary volume

### Step 1 — Take a full backup

```bash
confd_client db_backup_start db_name: exa_db backup_volume_name: ArchiveVolume1 level: 0 expire: 1w
```

### Step 2 — Gracefully shut down the database

```bash
confd_client db_stop db_name: DB1
```

### Step 3 — Stop the c4_cloud_command service for all nodes

```bash
sudo systemctl stop c4_cloud_command
```

### Step 4 — Add the new disks

Add the new disk to each node (physically or virtually install/attach).

### Step 5 — Power the servers back on

Ensure the nodes are fully back online and the c4 services plus the database is online.

### Step 6 — Connect to the COS container

```bash
c4 connect -i <PLAY_ID> -s cos
```

### Step 7 — Stop the database before enabling the new disk

```bash
confd_client db_stop db_name: DB1
```

### Step 8 — Register the new disk

```bash
confd_client st_device_add node: 11 devname: /dev/sdc disk: temporary
```

### Step 9 — Configure the temporary volume of the database to use the new disk by adding a parameter -temp_hdd_type=temporary to the database

```bash
confd_client db_configure db_name: Exasol params: '[-temp_hdd_type=temporary]'
```

### Step 10 — Start the database

```bash
confd_client db_start db_name: DB1
```

### Step 11 — Verify the temporary disk is being used in the database

```bash
csinfo -v
```

```bash
confd_client db_info db_name: exa_db
```

## Canonical references

- `documents/cos/confd-storage-devices.md` (`st_device_add`)
- `documents/cos/confd-backup-and-restore.md` (`db_backup_start`)
