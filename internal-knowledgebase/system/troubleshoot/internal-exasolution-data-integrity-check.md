---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Internal: EXASolution data integrity check"
summary: "Use checkdata to analyze storage-level integrity issues when database startup fails with persistent storage errors."
---
# Internal: EXASolution data integrity check

## Problem

Database startup fails and `PddServer` aborts/coredumps with persistent storage errors (for example missing blocks).

## Typical symptoms

```text
PersistentStorageError: Block does not exist ...
Internal error ... atomic-commit. Abort now.
Program received signal 6 (SIGABRT)
```

## Critical prerequisites

- Database must be offline before using `checkdata`.
- Run in controlled maintenance window.
- `remove_phantom_blocks` is high risk and may leave DB non-startable in some cases.

## Tool usage

Example binary location (version-specific):

```text
/usr/opt/EXASuite-5/EXASolution-5.0.14/bin/checkdata
```

Usage summary:

```text
checkdata -pers_volume_id <volume_id> -mode <mode> [-use_redundancy <n>]
```

Modes:

- `checksum`
- `data_integrity`
- `print_metadata`
- `remove_phantom_blocks`

## Recommended workflow

1. Run `data_integrity` first.
1. Check `missing blocks`, `phantom blocks`, and `critical blocks` columns.
1. Run `checksum` and `print_metadata` to collect additional evidence.
1. Use `remove_phantom_blocks` only as last resort and only with rollback/recovery plan.

Examples:

```shell
checkdata -pers_volume_id 0 -mode data_integrity
checkdata -pers_volume_id 0 -mode data_integrity -use_redundancy 2
checkdata -pers_volume_id 0 -mode checksum
checkdata -pers_volume_id 0 -mode print_metadata
```

Last resort:

```shell
checkdata -pers_volume_id 0 -mode remove_phantom_blocks
```

## Post-check step

After corrective operations and successful startup, refresh statistics:

```sql
ANALYZE DATABASE ESTIMATE STATISTICS;
```

## Reference

- SPOT-2189: database restart issues after phantom-block removal.

---

_We welcome feedback on this troubleshooting article._
