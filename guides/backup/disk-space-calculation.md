---
tool_name: confd_client
doc_type: reference
category: backup-restore
technical_entities:
  - backup
  - disk space
  - archive volume
summary: >
  How to calculate disk space requirements for Exasol backups — formulas for
  local and remote archive volumes with worked example.
---

# Backup Disk Space Calculation

## Remote Backups

```
(Full backup size × (num_full + 1)) + (Incremental size × num_incremental)
= Required space
```

## Local Backups

```
((Full backup size × (num_full + 1)) + (Incremental size × num_incremental)) × 2
= Required space
```

The `×2` accounts for redundancy. The `+1` provides headroom during creation.

## Typical Cycle

Sunday full backup (10-day retention) + Monday–Saturday incremental (3-day
retention).
