---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "SDDC enlargement (v7)"
summary: "Troubleshooting and execution flow for enlarging a v7 SDDC cluster while avoiding data-loss flags and node-order issues."
---
# SDDC enlargement (v7)

## Scope

This procedure applies to Exasol version 7 SDDC environments.

## Prerequisites

- SDDC architecture is understood.
- Valid backup is available.
- All required nodes are online and configured.
- No active hardware incidents.
- No target database has the `create new db` flag set.

## Critical Safety Check

Before enlargement, verify that no target database has `create new db` enabled:

```shell
dwad_client list | grep create
```

If this flag is present on a database to be enlarged, stop immediately and remove the flag first. Continuing can lead to data loss.

## Procedure

1. Add and activate new nodes in the cluster.
1. Add unused disks to EXAStorage for the target setup.
1. Stop the target database and wait until it is fully offline.
1. Enlarge the data volume first (archive volume after database enlargement).
1. Add nodes in strict SDDC order: first `SITE_A`, then `SITE_B`.
1. Start and enlarge the first side.
1. Stop the first side again after enlargement startup completes.
1. Add new nodes as reserve nodes on the second side and run enlargement there.
1. If enlargement reports expected transient failure but database start works, continue with the documented sequence and stop again.
1. Start the first side again.
1. Run database reorganization:

```sql
REORGANIZE DATABASE;
```

## Validation

- Both sites start cleanly.
- Node assignments are consistent with SDDC order.
- No active enlargement errors remain in logs.
- Reorganization completes successfully.

## Escalation

Escalate with logs if:

- Enlargement fails persistently on either site.
- Node-count mismatch or startup instability remains.
- Reorganization cannot be completed.

---

_We welcome feedback on this troubleshooting article._
