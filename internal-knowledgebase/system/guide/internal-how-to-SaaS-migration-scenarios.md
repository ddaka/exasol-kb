---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "Internal - SaaS migration scenarios"
summary: "Decision guide for selecting SaaS migration path based on source/target node topology and environment alignment."
---
# Internal - SaaS migration scenarios

## Purpose

Select the correct migration strategy before executing detailed SaaS migration runbooks.

## Scenario 1: Source in AWS and matching active-node count

Preferred path when source and target topology align.

High-level flow:

1. Validate version/parameter compatibility (including v8 TLS/encryption expectations).
2. Ensure source system is updated and stable.
3. Prepare SaaS account access and admin permissions for migration operators.
4. Create target SaaS database.
5. Provision S3 archive in SaaS environment and grant R/W access.
6. Attach S3 as remote archive on source cluster.
7. Validate connectivity (`sdfs add`/`sdfs get`).
8. Transfer required backup chain (L0 + incrementals).
9. Restore backup to SaaS target.
10. Complete SaaS-managed handover and post-migration tasks (users, BucketFS, drivers, clients).

Detailed execution runbook:

- `system/guide/db-migration-to-saas.md`

## Scenario 2: Source has fewer nodes than target

Use a staging cluster to enlarge topology before final migration to SaaS target.

## Scenario 3: Source has more nodes than target

Use a staging cluster/EXAmigration path to adjust topology before final migration.

## Notes

- Node-count mismatch scenarios require separate planning and validation windows.
- Always validate backup/restore path and post-restore SaaS control-plane requirements.


