---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Foreign backups not listed when backup objects are manually copied without <DB_NAME>/<BACKUP_ID>"
summary: "Explains why `db_backup_list ... show_foreign: True` can return an empty list although backup files exist on S3/SDFS, and how to fix the archive path structure."
---

# Foreign backups not listed when backup objects are manually copied without `<DB_NAME>/<BACKUP_ID>`

## Overview

This article explains why the following command can return an empty list even though backup-looking objects exist on archive storage:

```bash
confd_client db_backup_list db_name: <DB_NAME> show_foreign: True
```

The most common root cause in this scenario is a manual copy to S3/archive storage that does not preserve the required Exasol backup hierarchy.

## Symptom pattern

1. Foreign backup listing is empty:

```bash
confd_client db_backup_list db_name: <DB_NAME> show_foreign: True
[]
```

2. Objects are visible on archive storage:

```bash
sdfs list <VOLUME>
...
level_0/node_0/backup_<DATE>
level_0/node_0/metadata_<DATE>
level_0/node_0/status_<DATE>
```

## Why this happens

`sdfs list <VOLUME>` shows raw objects/files that exist and are reachable from COS.

`db_backup_list ... show_foreign: True` does not list raw files. It discovers backup sets using the expected Exasol path structure and metadata grouping.

If backup files are copied manually without DB and backup ID prefixes, they are visible in SDFS but not discoverable as foreign backups.

## Required path layout for discovery

For ConfD foreign backup discovery, backup artifacts must be under:

```text
<DB_NAME>/<BACKUP_ID>/level_0/node_0/backup_<DATE>
<DB_NAME>/<BACKUP_ID>/level_0/node_0/metadata_<DATE>
<DB_NAME>/<BACKUP_ID>/level_0/node_0/status_<DATE>
```

Depending on cluster topology, there can be additional `node_*` paths and levels. The critical requirement is preserving `<DB_NAME>/<BACKUP_ID>/...`.

## Incorrect manual-copy layout (common failure)

The following layout is not sufficient for foreign backup discovery:

```text
level_0/node_0/backup_<DATE>
level_0/node_0/metadata_<DATE>
level_0/node_0/status_<DATE>
```

Without `<DB_NAME>/<BACKUP_ID>`, ConfD cannot correctly scope and group the backup set for the target database.

## Technical behavior (high-level)

1. ConfD scans archive volumes for candidates matching expected backup hierarchy.
2. It groups artifacts by `<DB_NAME>/<BACKUP_ID>`.
3. It validates required components (`backup_*`, `metadata_*`, `status_*`).
4. Only valid grouped candidates are returned by `db_backup_list`.

If grouping fails because prefixes are missing, the result can be `[]`.

## Verification steps

1. Check object paths:

```bash
sdfs list <VOLUME>
```

If paths start directly with `level_0/...` (not `<DB_NAME>/<BACKUP_ID>/level_0/...`), this is the issue.

2. Compare with a known-good backup path (if available):

```text
<DB_NAME>/<BACKUP_ID>/level_0/node_0/...
```

## Resolution

### Preferred

Use supported Exasol backup/migration tooling so the correct hierarchy is created automatically.

### If manual repair is required

1. Determine correct `<DB_NAME>`.
2. Determine appropriate `<BACKUP_ID>` for the backup set.
3. Move/copy objects to:

```text
<DB_NAME>/<BACKUP_ID>/level_0/node_0/
```

4. Ensure all required files/nodes are present for the backup topology.
5. Re-run:

```bash
confd_client db_backup_list db_name: <DB_NAME> show_foreign: True
```

## Prevention

1. Do not copy only `level_0/node_0/*` into volume root.
2. When migrating backups, preserve at least the entire `<DB_NAME>/<BACKUP_ID>/` subtree intact.
3. Add this structure requirement to all runbooks involving manual S3/archive copy.

## Quick decision table

| Observation | Meaning | Action |
|---|---|---|
| `db_backup_list ... show_foreign: True` returns `[]` | No discoverable backup sets found | Validate archive path hierarchy |
| `sdfs list` shows `level_0/node_0/*` at root | Backup files exist but are orphaned for discovery | Move under `<DB_NAME>/<BACKUP_ID>/...` |
| Backups appear after path fix | Root cause confirmed | Update migration/runbook process |
