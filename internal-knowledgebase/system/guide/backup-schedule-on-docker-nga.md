---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "Configure Backup Schedules on Docker/NGA"
summary: "Legacy EXAoperation/NGA guide for configuring remote backup volumes and creating, modifying, and removing backup schedules via exaconf or ConfD API."
---

# Configure Backup Schedules on Docker/NGA

## Overview

This guide describes two ways to manage database backup schedules on legacy Docker/NGA deployments:

- COS CLI (`exaconf`)
- ConfD XML-RPC API (`db_backup_*` jobs)

It also includes validation and troubleshooting guidance for remote archive volumes.

## Prerequisites

- Legacy Docker/NGA environment is running.
- SSH access to COS.
- Existing database (for example `DB1`).
- Archive target available (local archive volume or remote volume such as S3/FTP/SMB/Azure Blob/GCS).

## Procedure

### 1. Add an archive or remote volume

Backups must not be stored on the data volume. Create an archive target first.

Example for a remote S3 volume:

```bash
exaconf add-remote-volume \
  --name s3_archive \
  --type s3 \
  --owner 500:500 \
  --id 10005 \
  --url https://<bucket>.s3-<region>.amazonaws.com/ \
  --username <access_key_id> \
  --passwd <secret_key> \
  --options cleanvolume
```

Commit configuration changes:

```bash
sed -i '/Checksum =/c\ Checksum = COMMIT' /exa/etc/EXAConf
exaconf commit
```

Validate remote volume access:

```bash
sdfs list 10005
```

### 2. Add backup schedules with COS CLI

Typical pattern:

- `L0` full backup on weekend
- `L1` incremental backups on weekdays

```bash
exaconf add-backup-schedule --backup-name "Full_Backup" \
  --db-name DB1 --volume 10005 --level 0 --expire "9d" \
  --minute "*" --hour 0 --day "*" --month "*" --weekday 6

exaconf add-backup-schedule --backup-name "Incremental_Backup" \
  --db-name DB1 --volume 10005 --level 1 --expire "3d" \
  --minute "*" --hour 0 --day "*" --month "*" --weekday 0,1,2,3,4,5

sed -i '/Checksum =/c\ Checksum = COMMIT' /exa/etc/EXAConf
exaconf commit
```

### 3. Add backup schedules with ConfD API

Equivalent API-based scheduling:

```python
conn.job_exec('db_backup_add_schedule', {
  'params': {
    'db_name': 'DB1',
    'backup_name': 'Full_Backup_API',
    'backup_volume_id': 10005,
    'level': 0,
    'expire': '9d',
    'minute': '*',
    'hour': '0',
    'day': '*',
    'month': '*',
    'weekday': '6',
    'enabled': True
  }
})

conn.job_exec('db_backup_add_schedule', {
  'params': {
    'db_name': 'DB1',
    'backup_name': 'Inc_Backup_API',
    'backup_volume_id': 10005,
    'level': 1,
    'expire': '3d',
    'minute': '*',
    'hour': '0',
    'day': '*',
    'month': '*',
    'weekday': '0,1,2,3,4,5',
    'enabled': True
  }
})
```

## Modify or Remove Schedules

### Remove via CLI

```bash
exaconf remove-backup-schedule --db-name DB1 --backup-name "Full_Backup"
exaconf remove-backup-schedule --db-name DB1 --backup-name "Incremental_Backup"
sed -i '/Checksum =/c\ Checksum = COMMIT' /exa/etc/EXAConf
exaconf commit
```

### Remove via ConfD API

```python
conn.job_exec('db_backup_remove_schedule', {
  'params': {
    'db_name': 'DB1',
    'backup_name': 'Full_Backup_API'
  }
})
```

### Modify via CLI

```bash
exaconf modify-backup-schedule --db-name DB1 --backup-name "Full_Backup" \
  --minute 30 --hour 20 --day "*" --month "*" --weekday 0
sed -i '/Checksum =/c\ Checksum = COMMIT' /exa/etc/EXAConf
exaconf commit
```

### Modify via ConfD API

```python
conn.job_exec('db_backup_modify_schedule', {
  'params': {
    'db_name': 'DB1',
    'backup_name': 'Full_Backup_API',
    'minute': '30',
    'hour': '20',
    'weekday': '0'
  }
})
```

## Verification

- Check `EXAConf` backup sections under `[DB : <db_name>]`.
- List backup files on the target volume:

```bash
sdfs list <volume_id>
```

- Track backup activity in logs:

```bash
ls -rt /exa/logs/db/DB1/ | grep PddServer | tail -1 | xargs tail -20f
```

## Troubleshooting

### Remote volume connection fails

Enable verbose remote-volume option in `EXAConf` (for example `cleanvolume,verbose`), commit, then re-run `sdfs list <volume_id>`.

Interpret common HTTP errors from verbose output:

- `403 Forbidden`: credentials or permissions are invalid.
- `404 Not Found`: endpoint or bucket URL is incorrect.

### Useful remote-volume options

- `cleanvolume`: remove expired remote backups.
- `nocompression`: write uncompressed backup data.
- `noverifypeer`: disable TLS peer verification (use only for diagnostics).
- `timeout=<seconds>`: increase client timeout for slow endpoints.

## References

- [Connect to ConfD with Python 3 (XML-RPC)](connecting-to-confd-with-python3.md)
- [confd_client — Backup and Restore](../../../cos/confd-backup-and-restore.md)
