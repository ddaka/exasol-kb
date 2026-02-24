---
tool_name: internal-knowledgebase
doc_type: reference
category: database-sql
title: "Estimate Zonemap Migration Time on Partitioned Tables"
summary: "Reference query for estimating startup-time impact of zonemap creation on partition columns after upgrade from versions without automatic partition zonemaps."
---

# Estimate Zonemap Migration Time on Partitioned Tables

## Context

On versions where zonemaps are created automatically for partition keys, upgrades from older releases may trigger zonemap creation during database startup.

Scope notes:

- Applies to partition columns.
- `HASHTYPE` and `DOUBLE` partition columns are excluded from zonemap support in this context.

## Purpose

Estimate startup-time impact caused by initial zonemap creation on eligible partition columns.

## Estimation Query

The following query uses a conservative throughput assumption of `200 MB/s` raw size per process.

```sql
SELECT
    SUM(ZEROIFNULL(data_migration_seconds)) AS migration_seconds
FROM (
    SELECT
        CAST(SUM(csz.raw_object_size) / 1024 / 1024 / 200 / NPROC() AS DEC(18,1)) AS data_migration_seconds
    FROM EXA_DBA_COLUMN_SIZES csz
    JOIN EXA_DBA_COLUMNS cols
      ON csz.column_object_id = cols.column_object_id
    WHERE cols.column_type NOT LIKE 'HASH%'
      AND cols.column_type <> 'DOUBLE'
      AND cols.column_partition_key_ordinal_position IS NOT NULL
);
```

If storage is faster than the conservative assumption, real migration time can be lower.

## Startup Behavior Control

If zonemap creation on startup must be deferred after upgrade, options include:

1. Remove partition keys from affected tables.
2. Start database with:

```text
-zonemapCreationOnPartitionedColumnsUponMigration=0
```

## Caution

Disabling zonemap creation can affect query performance on partitioned access paths until zonemaps are created later.
