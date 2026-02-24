---
tool_name: internal-knowledgebase
doc_type: guide
category: database-sql
title: "Playbook: absolute performance analysis for Exasol queries"
summary: "Investigate slow queries using EXA_SQL metrics and profile patterns to identify optimizer and data-shape bottlenecks."
---
# Playbook: absolute performance analysis for Exasol queries

## Purpose

Use this playbook when a query is objectively slow (not just slower than its own baseline) and you need to identify dominant execution bottlenecks.

## Step 1: Check query-level resource metrics

Start with aggregate runtime metrics:

```sql
SELECT *
FROM EXA_SQL_LAST_DAY
WHERE session_id = ?
  AND stmt_id = ?;
```

Focus areas:

- `CPU / RESOURCES` ratio: low ratio can indicate waits (disk/network/locks) or low-parallel sections.
- `HDD_READ` / `HDD_WRITE`: high first-run I/O may indicate cache misses or insufficient memory residency.
- `TEMP_DB_RAM_PEAK`: high temporary RAM often indicates heavy subselect materialization.

Rule-of-thumb (adjust to environment):

- Single query TEMP usage should usually remain below ~10% of DB RAM.
- Total TEMP pressure should usually remain below ~20%.

## Step 2: Analyze profile structure

Use profile data to locate where row volume or execution time explodes.

### Common pattern A: Expression index build

Signals:

- `PART_NAME = 'INDEX BUILD'`
- `PART_INFO` contains `EXPRESSION INDEX`

Cause:

- Join keys involve expressions/casts instead of aligned columns.

Actions:

- Harmonize join-column data types.
- Remove expressions from join predicates.
- Rewrite predicates so indexed plain columns are join keys.

### Common pattern B: Node sync imbalance

Signals:

- High `NODE SYNC` contribution in profile.

Cause:

- Skewed distribution or filters causing uneven per-node workload.

Actions:

- Validate distribution keys and high-null skew.
- Pre-filter dimensions before large joins.
- Consider controlled materialization (`ORDER BY FALSE`) to improve local join behavior.

### Common pattern C: Between/range join explosion

Signals:

- Join parts with very high `OUT_ROWS`, followed by filters collapsing rows.

Cause:

- Non-equality predicates producing large intermediate candidate sets.

Actions:

- Convert logic toward equality joins where possible.
- Reduce lookup relation before join.

### Common pattern D: Unfiltered subselect aggregation

Signals:

- Large subselect materialization, then heavy row drop in downstream join/filter.

Cause:

- Aggregation done before relevant filters are applied.

Actions:

- Push filters into subselects.
- Reorder logic to reduce input rows before aggregation.

### Common pattern E: Root scan filter

Signals:

- `PIPE SCAN` with near-total rows, then strong `PIPE FILTER` reduction.

Cause:

- Missing or non-used local index on literal filter column.

Action:

```sql
ENFORCE LOCAL INDEX ON <table>(<column>);
```

Validate with explain/profile; optimizer may still pick scan at high hit-rate.

### Common pattern F: UNION/UNION ALL materialization

Signals:

- Expensive union materialization before downstream joins.

Actions:

- Prefer `UNION ALL` over `UNION` when deduplication is unnecessary.
- Simplify union branches to unlock optimizer union-wrapper behavior.
- Push filters into branch inputs early.

## Step 3: Prioritize fixes

1. Remove avoidable row explosion first.
1. Reduce expensive materialization second.
1. Address skew and distribution third.
1. Scale hardware only after query-shape fixes are exhausted.

## Related playbooks

- `playbook-performance-query-relative.md`
- `playbook-performance.md`


