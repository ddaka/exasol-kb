---
tool_name: internal-knowledgebase
doc_type: reference
category: database-sql
title: "$EXA_SYS_VIEWS: DDL and hidden-object mapping"
summary: "Reference for using $EXA_SYS_VIEWS to inspect DDL behind SYS and EXA_STATISTICS views."
---
# $EXA_SYS_VIEWS: DDL and hidden-object mapping

## Purpose

`"$EXA_SYS_VIEWS"` exposes view definitions (DDL text) for system/statistics views and helps map visible views to hidden underlying objects.

This is useful for internal diagnostics and dependency analysis.

## Important limitations

- Hidden objects are undocumented and not part of a stable public interface.
- Names/structures can change between releases.
- Do not build customer-facing solutions on hidden objects.

## Covered schemas

- `SYS`
- `EXA_STATISTICS`
- `$ODBCJDBC`

## Basic query

```sql
SELECT *
FROM "$EXA_SYS_VIEWS"
WHERE view_name = 'EXA_DBA_AUDIT_SQL';
```

Inspect `VIEW_TEXT` to see the view definition and referenced underlying objects.

## Example dependency chain

```text
EXA_DBA_AUDIT_SQL
  -> $EXA_AUDIT_SQL
     -> EXA_STATISTICS."$EXA_STATS_AUDIT_SQL"
     -> SYS.EXA_COMMAND_IDS
     -> EXA_STATISTICS."$EXA_STATS_AUDIT_SQL_RESOURCES"
```

Use recursive inspection of referenced objects to understand dependency hierarchy.

## Reference

- <https://docs.exasol.com/db/latest/sql_references/system_tables.htm>


