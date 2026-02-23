---
tool_name: confd_client
doc_type: concept
category: backup-restore
technical_entities:
  - backup
  - archive volume
  - local archive volume
  - remote archive volume
  - backup expiration
summary: >
  Backup essentials — backup types (full level 0 through incremental level 9),
  local vs remote archive volumes, and backup expiration behaviour.
---

# Backup Essentials

Backups capture the entire database (not individual schemas/tables) at the time
the backup was started. Only committed transactions are included. Backups run in
the background with minimal performance impact.

> **Important:** Do not stop the database or modify archive/data volumes while a
> backup is in progress.

## Backup Types

| Level   | Type        | Description                                            |
|---------|-------------|--------------------------------------------------------|
| Level 0 | Full        | Contains all database blocks                           |
| Level 1 | Incremental | Changes since last Level 0 backup                     |
| Level 2 | Incremental | Changes since last Level 1 backup                     |
| ...     | ...         | Each level is incremental to the previous level        |
| Level 9 | Incremental | Changes since last Level 8 backup                     |

## Archive Volumes

| Type   | Speed | Fail Safety | Restore Types                                  |
|--------|-------|-------------|------------------------------------------------|
| Local  | Fast  | Within cluster | Blocking, non-blocking, virtual access      |
| Remote | Slower | Better (survives cluster crash) | Blocking only          |

Remote volume protocols: FTP/FTPS, SMB, Amazon S3, WebHDFS, Azure Blob Storage,
Google Cloud Storage.

## Backup Expiration

**Local volumes:** Expired backups are removed after a new backup completes, or
during backup if space runs out.

**Remote volumes:** Expired backups are kept by default unless `cleanvolume` is
set on the volume.
