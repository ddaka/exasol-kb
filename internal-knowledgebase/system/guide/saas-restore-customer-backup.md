---
tool_name: cos
doc_type: guide
category: system
title: "SaaS - Restore customer backup"
summary: "Restore SaaS customer databases from local or remote snapshot backups using controlled support workflow."
---
# SaaS - Restore customer backup

## Purpose

Perform support-driven restore requests for customer databases from snapshot backups.

Only the account owner can request restore operations.

## Prerequisites

- Valid backups available on local and/or remote snapshot volumes.
- Customer approval and downtime communication completed.
- Confirm snapshot volume IDs.

Get volume IDs (`SnapshotVolume` local, `SnapshotVolumeSync` remote):

```bash
csinfo
```

## Scenario A: restore from local snapshot

1. Stop database:

```bash
dwad_client stop <DB_NAME>
```

2. List local snapshots and choose restore target:

```bash
sbfs list <SNAPSHOT_VOLUME_ID>
```

3. Start DB in restore mode:

```bash
dwad_client pdd-restore <DB_NAME>
```

4. Restore selected snapshot:

```bash
dwad_client snapshot-restore <DB_NAME> <SNAPSHOT_VOLUME_ID> <BACKUP_NAME>
```

5. Remove invalid subsequent remote snapshots created after restore point:

```bash
sbfs remove <REMOTE_SNAPSHOT_VOLUME_ID> -name <INVALID_BACKUP_NAME>
```

6. Finalize restore and unblock remote sync:

```bash
sbfs finalize-restore <SNAPSHOT_VOLUME_ID> <REMOTE_SNAPSHOT_VOLUME_ID>
```

## Scenario B: restore from remote snapshot backup

1. Stop database:

```bash
dwad_client stop <DB_NAME>
```

2. List remote snapshots and select range:

```bash
sbfs list <REMOTE_SNAPSHOT_VOLUME_ID>
```

3. Prepare sync-back on local snapshot volume:

```bash
sbfs prepare-sync-back <SNAPSHOT_VOLUME_ID> -data-volume <DATA_VOLUME_ID> -metadata-volume <METADATA_VOLUME_ID>
```

4. Sync requested backup range from remote to local:

```bash
sbfs sync-back <REMOTE_SNAPSHOT_VOLUME_ID> <SNAPSHOT_VOLUME_ID> -range <FIRST_ID> <LAST_ID>
```

5. Continue with Scenario A from local snapshot restore step 2.

## Warnings

- `snapshot-restore` invalidates later snapshots by design.
- Always run `sbfs finalize-restore` after cleanup, otherwise remote sync remains blocked.

## Canonical references (de-duplication)

- `documents/cos/confd-backup-and-restore.md`
- `documents/internal-knowledgebase/system/guide/saas-snapshot-backup-restore.md`
