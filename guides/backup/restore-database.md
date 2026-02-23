---
tool_name: confd_client
doc_type: guide
category: backup-restore
subcommands:
  - db_restore
  - db_stop
  - db_backup_progress
technical_entities:
  - restore
  - blocking restore
  - non-blocking restore
  - virtual access
summary: >
  How to restore an Exasol database from backup — prerequisites, backup ID
  format, db_restore command with blocking/nonblocking/virtual access restore
  types, and verification via EXA_SYSTEM_EVENTS.
---

# Restore Database from Backup

## Prerequisites

- Database must be stopped
- Same number of active nodes as the backup source
- Same LTS version as the backup version

## Backup ID Format

```
VOLUME_ID BACKUP_PATH BACKUP_DATABASE_NAME
```

Example: `10001 MY_DATABASE/id_1/level_0/node_0/backup_202406131208 MY_DATABASE`

## Procedure

```bash
confd_client db_stop db_name: MY_DATABASE

confd_client db_restore \
  db_name: MY_DATABASE \
  backup_id: '10001 MY_DATABASE/id_1/level_0/node_0/backup_202406131208 MY_DATABASE' \
  restore_type: blocking
```

| Restore Type     | Database Available | Source           |
|------------------|--------------------|------------------|
| `blocking`       | No                 | Local or remote  |
| `nonblocking`    | Yes (background)   | Local only       |
| `virtual access` | Yes (on-demand)    | Local only       |

## Monitor Restore

```bash
confd_client db_backup_progress db_name: MY_DATABASE
```

Database starts automatically when restore completes.

## Verification

```sql
SELECT MEASURE_TIME, EVENT_TYPE
FROM EXA_SYSTEM_EVENTS
ORDER BY MEASURE_TIME DESC;
```

Look for `RESTORE_START` and `RESTORE_END` events.
