---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "Saas - Snapshot Backup Restore"
summary: "Specialized runbook for cross-bucket snapshot sync and restore; use with the main SaaS customer restore workflow."
---
# Saas - Snapshot Backup Restore

## Purpose

Use this workflow when snapshots must be synchronized between S3 buckets before restore.

## De-duplication note

The primary restore workflow is documented in:

- `documents/internal-knowledgebase/system/guide/saas-restore-customer-backup.md`

This page only keeps the cross-bucket sync specifics.

## Cross-bucket preparation steps

1. Get instance role ARN from master deployment.
2. In destination/offline account, grant source role access to target S3 bucket.
3. Pause snapshot automation (`sbfs` process / cron jobs in `/etc/crontab.d/exasnapshotbackups`).
4. Stop DB before data movement:

```bash
dwad_client stop Exasol
```

5. Clean destination snapshot folders:

```bash
aws s3 rm --profile <profile> s3://<new-bucket>/volumes/3 --recursive
aws s3 rm --profile <profile> s3://<new-bucket>/volumes/4 --recursive
```

6. Sync snapshot volumes from source bucket:

```bash
aws s3 sync --profile <profile> s3://<old-bucket>/volumes/3 s3://<new-bucket>/volumes/3
aws s3 sync --profile <profile> s3://<old-bucket>/volumes/4 s3://<new-bucket>/volumes/4
```

7. Verify and restore through standard flow:

```bash
sbfs verify-dataintegrity 4
sbfs sync-back 4 2 -range <id-from> <id-to>
dwad_client pdd-restore Exasol
sbfs list 2
dwad_client snapshot-restore Exasol 2 <snapshot-id-string>
```

## Post-restore expectation

Database state should return to running/healthy and snapshot automation should be re-enabled.
