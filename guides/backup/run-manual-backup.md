---
tool_name: confd_client
doc_type: guide
category: backup-restore
subcommands:
  - db_backup_start
  - db_backup_progress
  - db_backup_abort
technical_entities:
  - backup
  - manual backup
summary: >
  How to run a manual backup in Exasol — db_backup_start parameters, monitor
  progress with db_backup_progress, and abort with db_backup_abort.
---

# Run Manual Backup

**Prerequisites:** Database must be running. Archive volume must exist.

## Start Backup

```bash
confd_client db_backup_start \
  db_name: MY_DATABASE \
  backup_volume_name: r0001 \
  level: 0 \
  expire: '1w'
```

| Parameter             | Description                                  |
|-----------------------|----------------------------------------------|
| `db_name`             | Database name                                |
| `backup_volume_name`  | Archive volume for the backup                |
| `level`               | Backup level (0 = full, 1–9 = incremental)   |
| `expire`              | Expiration time (`#w #d` format)             |

## Monitor Progress

```bash
confd_client db_backup_progress db_name: MY_DATABASE
```

Returns percentage (100 = complete).

## Abort Backup

```bash
confd_client db_backup_abort db_name: MY_DATABASE
```
