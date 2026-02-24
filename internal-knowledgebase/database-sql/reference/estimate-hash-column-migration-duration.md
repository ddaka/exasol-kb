---
tool_name: internal-knowledgebase
doc_type: reference
category: database-sql
title: "Estimate Hash Column Migration Duration (v7.1.8)"
summary: "Reference SQL to estimate startup-time impact from hash-column format migration and related index rebuild operations in Exasol 7.1.x upgrades."
---

# Estimate Hash Column Migration Duration (v7.1.8)

## Context

In Exasol 7.1.7/7.1.8, hash data format changes can increase startup time during upgrade because hash columns (and affected indexes) may require migration/rebuild.

Relevant changelog reference:

- <https://exasol.my.site.com/s/article/Changelog-content-12960?language=en_US>

## Estimation Goal

The query below estimates total startup impact in seconds, split into:

- data migration time
- index rebuild time

The estimate is intended to be conservative, but real-world deviations are possible.

## Assumptions

- License capacity can accommodate largest migrated hash object and related index build.
- Storage throughput is sufficient (roughly >= 300 MB/s read/write).
- Network is not the bottleneck relative to storage throughput.

## Estimation Query

```sql
SELECT
    ZEROIFNULL(SUM(overall_seconds)) AS overall_seconds,
    ZEROIFNULL(SUM(data_migration_seconds)) AS data_migration_seconds,
    ZEROIFNULL(SUM(index_rebuild_seconds)) AS index_rebuild_seconds
FROM (
    SELECT
        column_schema,
        column_table,
        ZEROIFNULL(data_migration_seconds) AS data_migration_seconds,
        ZEROIFNULL(index_rebuild_seconds) AS index_rebuild_seconds,
        ZEROIFNULL(data_migration_seconds) + ZEROIFNULL(index_rebuild_seconds) AS overall_seconds
    FROM (
        SELECT
            column_schema,
            column_table,
            CAST(2 * SUM(raw_object_size) / 1024 / 1024 / 200 / NPROC() AS DEC(18,1)) AS data_migration_seconds
        FROM "$EXA_COLUMN_SIZES"
        WHERE column_type LIKE 'HASH%'
        GROUP BY 1,2
    ) columns
    FULL OUTER JOIN (
        SELECT
            index_schema AS idx_schema,
            index_table AS idx_table,
            CAST(SUM(raw_object_size) / 1024 / 1024 / 80 / NPROC() AS DEC(18,1)) AS index_rebuild_seconds
        FROM "$EXA_INDICES"
        WHERE index_object_id IN (
            SELECT DISTINCT ic.index_object_id
            FROM "$EXA_INDEX_COLUMNS" ic
            JOIN EXA_DBA_COLUMNS tc
              ON ic.index_schema = tc.column_schema
             AND ic.index_table = tc.column_table
             AND ic.column_name = tc.column_name
            WHERE tc.column_type LIKE 'HASH%'
        )
        GROUP BY 1,2
    ) indices
      ON columns.column_schema = indices.idx_schema
     AND columns.column_table = indices.idx_table
);
```

## Notes

- Use this estimate for planning maintenance windows, not as an SLA-grade predictor.
- Re-check on production-like hardware if precise cutover timing is required.
