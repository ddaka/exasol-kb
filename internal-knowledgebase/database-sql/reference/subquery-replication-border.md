---
tool_name: internal-knowledgebase
doc_type: reference
category: database-sql
title: "Subquery replication border"
summary: "Reference for the undocumented -subqueryreplicationborder setting used for temporary subquery/materialization replication decisions."
---
# Subquery replication border

## Purpose

`-soft_replicationborder_in_numrows` controls replication decisions for persistent tables.

For temporary subquery/materialized objects, Exasol uses a separate (undocumented) parameter:

```text
-subqueryreplicationborder
```

## Key behavior

- Default value is `100000` rows.
- Affects optimizer decision whether temporary subquery results are replicated.
- Applies to subquery/materialization objects, not persistent table replication threshold logic.

## Risk and cautions

- Increasing this value can increase DB RAM usage.
- Excessive values can degrade overall system performance under concurrent load.
- Parameter is undocumented; behavior is not guaranteed as a public contract.

## Configuration

Set via extra DB parameters:

```text
-subqueryreplicationborder=<numrows>
```

## Version notes

- In Exasol 8, `-soft_replicationborder_in_numrows` is typically managed via `ALTER SYSTEM`.
- Equivalent `ALTER SYSTEM` handling is not available for `-subqueryreplicationborder`; DB parameter path is used.

## Related references

- <https://docs.exasol.com/db/latest/performance/best_practices.htm>
- <https://exasol.my.site.com/s/article/Replication-border-in-Exasol-6-1>
- <https://exasol.my.site.com/s/article/Changelog-content-16000>


