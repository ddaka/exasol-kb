---
tool_name: internal-knowledgebase
doc_type: guide
category: database-sql
title: "Playbook: differential performance analysis for Exasol queries"
summary: "Compare fast vs slow executions of similar SQL to identify environmental or plan-level causes of performance degradation."
---
# Playbook: differential performance analysis for Exasol queries

## Purpose

Use differential analysis when a query became slower than before and you need to explain the change.

## Inputs required

- Session/statement identifiers for slow and fast executions.
- SQL text variants (if available).
- Profile data for both runs (preferred).
- Optional logs for deeper comparison.

## Step 1: Compare query-level metrics

```sql
SELECT *
FROM EXA_SQL_LAST_DAY
WHERE session_id = ?
  AND stmt_id = ?;
```

For both executions, compare:

- `RESOURCES`: concurrency impact proxy.
- `CPU`: effective CPU usage relative to allocated resources.
- `HDD_READ` and read-size/duration metrics: out-of-memory data access impact.
- `TEMP_DB_RAM_PEAK`: materialization and intermediate footprint changes.

Derived indicators:

- CPU-time proxy: `duration * CPU`
- Network-work proxy: `duration * NET`

Large shifts suggest changed execution shape or row volume.

## Step 2: Compare profile plans

Retrieve profile for both executions:

```sql
SELECT *
FROM EXA_USER_PROFILE_LAST_DAY
WHERE session_id = ?
  AND stmt_id = ?
ORDER BY part_id;
```

Key checks:

- Join/order pipeline equality (`PART_NAME`, `PART_INFO`, `OBJECT_NAME`).
- `OBJECT_ROWS`, `IN_ROWS`, `OUT_ROWS` deltas.
- Presence/absence of expensive parts (index build, node sync, large temp subselects).

Interpretation:

- Different plan/order -> most likely main reason for runtime shift.
- Same plan/order but different row counts -> data-shape/filter/selectivity change.
- Same plan/rows but different durations -> environment effects (concurrency, I/O waits, locks).

## Step 3: Account for environment influences

### Concurrency

Lower `RESOURCES` usually means less CPU share due to concurrent workload.

### Disk pressure

Increased disk reads can lower effective CPU utilization and inflate runtime.

### Synchronization waits

Look for node imbalance indicators (for example high `NODE SYNC` contribution) when available.

## Step 4: Use log files when profile is insufficient

Logs can help when:

- Profile for one side is missing.
- Plan differences are unclear.
- Wait reasons are not visible in profile views.

Tooling:

- `sumup` from `exasol/dev-tools` can assist with cross-run log comparison.

## Step 5: Identify “same query” robustly

Exact SQL text matching is often unreliable because of dynamic literals, generated comments, and variant generation.

Use a combination of:

- Normalized SQL text matching.
- Profile-shape matching (tables/joins).
- Time-window and workload context.

## Related playbooks

- `playbook-performance-query-absolute.md`
- `playbook-performance.md`


