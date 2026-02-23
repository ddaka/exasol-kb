---
tool_name: confd_client
doc_type: guide
category: backup-restore
subcommands:
  - db_backup_list
technical_entities:
  - backup
  - Exasol Admin
summary: >
  How to list existing backups in Exasol — via Exasol Admin UI or
  confd_client db_backup_list, including foreign database backups.
---

# List Backups

## Exasol Admin

Open the **Backups** page — shows all backups across archive volumes.

## Command Line

```bash
confd_client db_backup_list db_name: MY_DATABASE
```

Include foreign database backups:

```bash
confd_client db_backup_list db_name: MY_DATABASE show_foreign: True
```
