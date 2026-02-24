---
tool_name: internal-knowledgebase
doc_type: reference
category: database-sql
title: "Big Table Join Restrictions Explained"
summary: "Reference for Exasol optimizer behavior on very large join tables, including big-table detection logic and penalty-cost effects."
---

# Big Table Join Restrictions Explained

## Overview

For very large joins, Exasol may change join strategy to avoid expensive index-swapping behavior.

Typical observations:

- A previously index-joined large table becomes a scan-side table.
- Queries involving extremely large join indexes run very slowly or stall under heavy swapping.

## Why This Happens

Exasol join execution relies on lookup indexes for join-side tables. If an index becomes too large relative to available DB RAM, swapping can dominate execution time.

To reduce this risk, the optimizer applies special handling for "big tables" by adding penalty costs to join plans that would use those tables as index-lookup join sides.

## Big Table Detection

Big-table classification is controlled by the optimizer parameter `-cboBigTableLimit` (default: `0.5`).

Interpretation with default value:

- If estimated join index size for a table is at least 50% of total DB RAM, it is treated as a big table.

## Identify Candidate Big Tables

Example query (default threshold `0.5`):

```sql
WITH startup AS (
  SELECT db_ram_size * 1024 AS db_ram_mb
  FROM exa_system_events
  WHERE event_type = 'STARTUP'
    AND measure_time = (
      SELECT MAX(measure_time)
      FROM exa_system_events
      WHERE event_type = 'STARTUP'
    )
), tables AS (
  SELECT table_schema, table_name, table_row_count
  FROM exa_all_tables
)
SELECT
  t.table_schema,
  t.table_name,
  t.table_row_count,
  0.5 AS big_table_parameter_value,
  s.db_ram_mb AS system_db_ram_mb,
  s.db_ram_mb * 0.5 AS big_table_limit_mb,
  (t.table_row_count * 20) / (1024 * 1024) AS estimated_index_mb,
  ((t.table_row_count * 20) / (1024 * 1024)) / s.db_ram_mb AS index_db_ram_ratio
FROM tables t
CROSS JOIN startup s
WHERE ((t.table_row_count * 20) / (1024 * 1024)) / s.db_ram_mb >= 0.5
ORDER BY index_db_ram_ratio DESC;
```

Adjust `0.5` if your environment uses a different `-cboBigTableLimit`.

## Optimizer Behavior for Big Tables

When a table is classified as big:

- Additional cost penalties are added to plans that use it as join-index side.
- Optimizer tends to prefer plans that scan the big table instead of indexing into it.
- In multi-join plans, big-table join placement is strongly discouraged unless alternatives are costlier.

## Tuning Notes

- Changing penalty-related behavior should be considered advanced tuning.
- Do not modify optimizer parameters without support guidance.
- In many cases, schema/model/query changes are safer than optimizer overrides.
