---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "INTERNAL: Rename an Existing Database Instance"
summary: "Operational runbook for renaming a database instance by creating a new instance shell and reattaching the original data volume safely."
---

# INTERNAL: Rename an Existing Database Instance

## Scope

This procedure renames a database instance by creating a new instance and reusing the original data volume.

## Preconditions

- Existing database is quiesced (no active workload).
- Full backup and rollback plan in place.
- Operators understand instance/volume mapping and startup flags.

## Procedure

1. Stop all activity and shut down the existing database.
2. Create a small dummy data volume.
   - Rule of thumb: ~`300 MB` per node.
3. Create a new database instance with the same technical configuration as the old one.
4. Attach dummy volume to the new instance.
5. Start the new instance once using the dummy volume.
6. Stop the new instance.
7. Reattach the original production data volume to the new instance.
   - Ensure `create new database` is **not** enabled.
8. Ensure old instance is stopped.
9. Start the new instance with the original data volume.
10. Validate application behavior and customer confirmation.
11. Delete old database instance.
12. Update monitoring references (if managed service monitoring applies).

## Notes

- New instance can use the same port as the old instance.
- Pay extra attention to startup flags and volume assignment to avoid accidental reinitialization.
