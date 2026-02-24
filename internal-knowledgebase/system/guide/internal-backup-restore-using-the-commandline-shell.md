---
tool_name: dwad_client
doc_type: guide
category: system
title: "Internal: Backup/restore using command line (shell)"
summary: "Internal command-line restore workflows using dwad_client for blocking, non-blocking, and virtual restore modes."
---
# Internal: Backup/restore using command line (shell)

## Purpose

Document internal `dwad_client` restore execution patterns from shell.

## Scope

Internal/legacy restore operations. For standard supported backup/restore flows, use canonical `confd_client` docs.

## Prerequisites

- Backup exists and path is known.
- Appropriate maintenance/change window.
- Required privileges in COS namespace.

## Non-blocking restore

```shell
dwad_client pdd-restore DB1
dwad_client check-restore-ready-state DB1
dwad_client storage-restore-nonblocking DB1 2 "DB1/id_1/level_0/node_0/backup_201805171035"
```

## Blocking restore

```shell
dwad_client pdd-restore DB1
dwad_client check-restore-ready-state DB1
dwad_client storage-restore DB1 2 "DB1/id_1/level_0/node_0/backup_201805171035"
```

## Virtual restore access

```shell
# Prepare dedicated data volume and VR database first.
dwad_client pdd-restore DB1
dwad_client check-restore-ready-state DB1
dwad_client storage-restore-virtual DB1 2 "DB1/id_1/level_0/node_0/backup_201805171035"
```

## De-duplication note

Canonical backup/restore command syntax for standard operations:

- `documents/cos/confd-backup-and-restore.md`


