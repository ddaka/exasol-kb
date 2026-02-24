---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "Choose Exasol disaster recovery option"
summary: "Operational guide to choose and implement an Exasol disaster recovery pattern based on RTO, RPO, and platform constraints."
---
# Choose Exasol disaster recovery option

## Purpose

Provide a practical runbook to select the right Exasol DR approach and define minimum implementation controls.

## Inputs required before decision

- Target `RTO` (maximum acceptable downtime).
- Target `RPO` (maximum acceptable data loss).
- Cloud/region constraints.
- Budget constraints (always-on standby vs on-demand recovery).
- Operational maturity (automation, monitoring, runbooks, drills).

## DR options and when to choose them

## 1) Backup-based recovery (baseline)

Choose when:

- Cost efficiency is primary.
- Longer RTO is acceptable.

Implementation minimum:

- Cross-zone/cross-region backup storage.
- Documented restore runbook.
- Periodic restore validation drills.

## 2) Cold standby cluster

Choose when:

- Lower RTO is required than backup-only.
- You can maintain a preconfigured standby environment.

Implementation minimum:

- Version/config parity between production and standby.
- Automated startup and restore workflow.
- Scheduled standby health checks.

## 3) Test/Dev as emergency target

Choose when:

- Budget is limited.
- Temporary service degradation during failover is acceptable.

Implementation minimum:

- Explicit switchover criteria.
- Backup sync and recovery workflow.
- Conflict policy for Test/Dev workload interruption.

## 4) ETL dual-write / replicated ingest

Choose when:

- Very low RPO is required.
- Data ingestion architecture supports parallel writes.

Implementation minimum:

- Source-of-truth consistency checks.
- Divergence detection and reconciliation process.
- Frequent failover tests.

## 5) Stretched or dual-data-center architectures

Choose when:

- You need aggressive availability targets and can support high complexity.

Implementation minimum:

- Proven network/storage design.
- Strict change control and failure testing.
- Vendor-supported topology validation.

## Required controls for any option

- DNS/application connection failover procedure.
- Restore of database-adjacent state: BucketFS artifacts, driver configs, platform settings, licenses/secrets.
- Monitoring and alerting for backup integrity and replication status.
- Documented failback strategy.

## Validation cadence

- Run at least one full DR exercise per quarter.
- Record measured RTO/RPO for each drill.
- Track remediation actions until closed.

## Reference

For detailed conceptual background, see:

- `system/reference/exasol-dr-options.md`


