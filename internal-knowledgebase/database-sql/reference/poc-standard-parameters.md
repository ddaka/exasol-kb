---
tool_name: internal-knowledgebase
doc_type: reference
category: database-sql
title: "PoC Standard Parameters"
summary: "Reference list of historical PoC startup parameters (mainly v6-era) with notes on replication, heap sizing, and platform-specific tuning caveats."
---

# PoC Standard Parameters

## Scope

This document captures historical PoC parameter sets used primarily for Exasol `v6.x` environments, plus legacy `v5` notes.

Use with caution:

- Many parameters are version- and workload-specific.
- Apply only with explicit support/performance-engineering guidance.

## Common v6.x PoC Set

```text
-useIndexWrapper=0
-disableIndexIteratorScan=1
-prestart=10
```

Often useful in specific workloads:

```text
-soft_replicationborder_in_numrows=10000000
```

Related background:

- <https://exasol.my.site.com/s/article/Replication-border-in-Exasol-6-1>

## Heap Sizing for High-Core/UDF-Heavy Workloads

```text
-maxProcessHeapMemory=8192
```

(or higher, e.g. `16384` depending on workload and memory budget)

If increased:

- Review `-maxSystemHeapMemory` accordingly.
- Rebalance DB RAM allocation to avoid overcommit.

## Azure-Specific Notes (historical)

```text
-keepalive=30
```

Historically evaluated options:

```text
-storageOptimizerSize=0
-storageLimitBorder=1024
-checkBlocksOnCommit=0
```

## Subquery Replication Tuning

For workloads with expensive global joins/materialized subselect patterns:

```text
-subqueryreplicationborder=<numrows>
```

Default historically around `100000` rows.

## Legacy v5 Notes (Outdated)

### Typical v5 PoC adjustments

| Parameter | Value | Purpose |
| --- | --- | --- |
| `-useIndexWrapper` | `0` | Disable IndexWrapper usage |
| `-vectorsize` | `2048` | Internal pipeline vector size |
| `-cacheMonitorSamplingRate` | `1` | Influence cache monitor sampling behavior |
| `-disableindexiteratorscan` | `1` | Disable sorted group-by path known to scale poorly on some high-core systems |

### Additional sometimes-used v5 options

| Parameter | Typical Value | Purpose |
| --- | --- | --- |
| `-disableviewoptimization` | `1` | Avoid repeated view materialization behavior |
| `-soft_replicationborder_in_numrows` | default `100000` | Raise row-count replication threshold |
| `-soft_replicationborder_in_kb` | default `1000000` | Raise size-based replication threshold |

## Diagnostics

To inspect effective command-line parameter values on a running system, query `EXA_COMMANDLINE`.

Related reference:

- [EXA_COMMANDLINE System Table](exa-commandline.md)
