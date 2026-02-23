---
tool_name: confd_client
doc_type: guide
category: backup-restore
subcommands:
  - db_restore
  - db_backup_list
  - st_volume_create
  - st_volume_delete
  - db_create
  - db_delete
  - db_stop
technical_entities:
  - virtual access
  - backup
  - restore
summary: >
  How to use virtual access to read specific objects from an Exasol backup
  without a full restore — DB RAM calculation, temporary volume and database
  creation, virtual restore, data import/export, and cleanup.
---

# Virtual Access on Backup

Virtual access lets you access specific objects from a backup without a full
restore. Virtual instances do not count toward the license limit.

## Prerequisites

- Local archive volume only
- Backup must not have an expire date
- Sufficient free DB RAM

## DB RAM Calculation

```
Number of active nodes × 4 GiB = DB RAM for virtual instance
```

## Setup Steps

### 1. Adjust DB RAM

If needed — stop DB, reduce `mem_size`, restart.

### 2. Create Temporary Data Volume

```bash
confd_client st_volume_create \
  name: VIRTUAL_DATA disk: disk2 type: data \
  size: '20 GiB' num_master_nodes: 4 \
  nodes: '[11, 12, 13, 14]' redundancy: 1 \
  owner: '[500, 500]'
```

### 3. Create Virtual Database Instance

Use a different port:

```bash
confd_client db_create \
  db_name: VIRTUAL_DB \
  data_volume_name: VIRTUAL_DATA \
  nodes: '[11, 12, 13, 14]' \
  num_active_nodes: 4 \
  port: 9563 \
  version: 8.26.0 \
  mem_size: '20 GiB'
```

### 4. Find Backup ID

```bash
confd_client db_backup_list db_name: MY_DATABASE
```

### 5. Start Virtual Restore

```bash
confd_client db_restore \
  db_name: VIRTUAL_DB \
  restore_type: 'virtual access' \
  backup_id: '9 MY_DATABASE/id_2/level_0/node_0/backup_202305081443 MY_DATABASE'
```

### 6. Import/Export Data

Connect to the virtual DB via SQL client and use `IMPORT`/`EXPORT` statements.

### 7. Clean Up

```bash
confd_client db_stop db_name: VIRTUAL_DB
confd_client db_delete db_name: VIRTUAL_DB
confd_client st_volume_delete vname: VIRTUAL_DATA
```
