---
tool_name: internal-knowledgebase
doc_type: reference
category: system
title: "Exasol Disaster Recovery Options"
summary: "Reference guide to Exasol disaster recovery patterns, backup models, restore modes, and recovery trade-offs."
---

# Exasol Disaster Recovery Options

## Purpose

This document summarizes disaster recovery (DR) options for Exasol and outlines the trade-offs between recovery time, recovery point, cost, and operational complexity.

## Core Terms

- `RTO` (Recovery Time Objective): Maximum acceptable service downtime.
- `RPO` (Recovery Point Objective): Maximum acceptable data loss window.

## Exasol Backup Model

- Backups are generated in parallel across database nodes.
- Backup levels:
  - `L0` full
  - `L1` differential
  - `L2`-`L9` incremental
- Higher levels depend on lower levels during restore.
- Backup files include committed database objects and metadata.
- Backups are not encrypted by default.

## Archive Volume Types

### Local Archive Volume

- Uses an SDFS-backed volume in the cluster.
- Optimized when backup-node and database-node placement is aligned.
- Supports automatic cleanup of expired backups.

### Remote Archive Volume

- Stores backups outside the cluster (for example S3, Azure Blob, GCS, SMB, FTP, WebHDFS).
- Retention, encryption, and redundancy depend on remote service configuration.
- Expired-backup cleanup is optional and configuration-dependent.

### Remote SDFS (Cross-Cluster)

- A local archive volume on another Exasol cluster can be used as remote backup target.
- Avoid double compression when writing compressed backups into SDFS-based remote targets.

## Restore Modes

### Blocking Restore

- Database is offline until restore is complete.

### Non-Blocking Restore

- Database starts while restore continues in the background.
- Data is fetched on demand for not-yet-restored segments.

### Virtual Access Restore

- Starts a temporary database on backup data.
- Useful for selective object recovery with `IMPORT`/`EXPORT`.

## DR Deployment Patterns

### ETL Dual-Write

- Two clusters ingest the same data stream.
- Improves failover readiness, but increases synchronization complexity.

### Backup-Based Recovery

- Backups are stored in another zone or region.
- Recovery requires provisioning target infrastructure plus restore execution.

### Cold Standby

- Preconfigured standby cluster kept ready for activation.
- Recovery typically includes cluster start plus restore.

### Test/Dev as Standby

- Reuses non-production environment as emergency target.
- Reduces infrastructure cost but requires strict runbooks.

### Snapshot-Assisted Recovery

- Infrastructure snapshots can speed environment recovery.
- Snapshots do not replace consistent database backup strategy.

### Stretched Storage / Dual Data Center

- Cross-site storage architecture can reduce RTO.
- Requires strict network, storage, and operational design.

## Recovery Scope Beyond Database Files

A DR plan should also include:

- EXAoperation and configuration state
- BucketFS buckets and artifacts
- Driver packages and integration settings
- Licenses and environment-specific secrets
- Database parameterization and authentication settings

## Decision Guidance

- Prioritize backup-based DR for robust baseline protection.
- Add cold standby where lower RTO is required.
- Use dual-write or stretched architectures only with proven operational maturity.

## References

- <https://docs.exasol.com/db/latest/administration/on-premise/backup_restore.htm>
- <https://docs.exasol.com/db/latest/planning/fail_safety.htm>
- <https://docs.exasol.com/db/latest/planning/business_continuity/sddc_details.htm>
