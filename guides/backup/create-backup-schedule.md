---
tool_name: confd_client
doc_type: guide
category: backup-restore
subcommands:
  - db_backup_add_schedule
  - st_volume_list
  - st_volume_info
  - remote_volume_list
  - remote_volume_info
  - db_info
technical_entities:
  - backup schedule
  - archive volume
  - Exasol Admin
summary: >
  How to create backup schedules in Exasol — via Exasol Admin UI or
  confd_client db_backup_add_schedule with weekly full + daily incremental
  example.
---

# Create Backup Schedule

Available via **Exasol Admin** (2025.1+) or command line.

## Exasol Admin

1. Open the **Schedules** page
2. Click **+** to add a schedule
3. Enter: Name, Database, Volume, Level, Frequency, Expiration (optional)
4. Click **Add**

## Command Line

### Find Archive Volume Info

```bash
# Local volumes
confd_client st_volume_list | grep name
confd_client st_volume_info vname: VOLUME_NAME

# Remote volumes
confd_client remote_volume_list
confd_client remote_volume_info remote_volume_name: VOLUME_NAME
```

### Create Schedule

Typical weekly schedule (full Sunday + incremental Mon–Sat):

```bash
confd_client db_backup_add_schedule \
  db_name: MY_DATABASE \
  backup_name: weekly_full_backup \
  backup_volume_name: VOLUME_NAME \
  enabled: true \
  level: 0 \
  expire: '1w 3d' \
  minute: 0 hour: 0 day: "'*'" month: "'*'" weekday: 0

confd_client db_backup_add_schedule \
  db_name: MY_DATABASE \
  backup_name: daily_incremental \
  backup_volume_name: VOLUME_NAME \
  enabled: true \
  level: 1 \
  expire: '3d' \
  minute: 0 hour: 0 day: "'*'" month: "'*'" weekday: "'1,2,3,4,5,6'"
```

## Verification

```bash
confd_client db_info db_name: MY_DATABASE
```

Backup schedules appear in the `config: backups:` section.
