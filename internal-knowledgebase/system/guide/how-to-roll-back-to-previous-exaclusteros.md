---
tool_name: cos
doc_type: guide
category: system
title: "How to roll back to previous EXAClusterOS"
summary: "Emergency rollback procedure when EXAClusterOS upgrade fails before databases are started on the new version."
---
# How to roll back to previous EXAClusterOS

## Purpose

Recover from failed EXAClusterOS upgrade when rollback is still possible.

## Rollback boundary conditions

Rollback is supported only if both are true:

- No database has been started on the upgraded/newer version.
- Data nodes are in suspended state before management-node reboot.

## Emergency rollback procedure

1. Start `n10` (management node), ideally isolated from data-node network links.
2. From virtual console, revert version references to previous known-good version in:
   - `/etc/cos.conf`
   - `/root/.bashrc`
   - `/etc/init.d/cos`
3. Re-enable network links to data nodes.
4. Reboot management node.
5. Validate cluster control-plane health before resuming normal operations.

## Validation checklist

- Management services start cleanly.
- Data nodes reconnect without version mismatch errors.
- Database start is possible on previous version.

## Notes

- This is an emergency path and must be executed by experienced operators.
- Keep full incident timeline and config diffs for postmortem.

## References

- <https://docs.exasol.com/administration/on-premise/updates.htm>
- <https://downloads.exasol.com/>
- Canonical COS update command reference: `documents/cos/confd-system-and-infrastructure.md` (`update_system`)


