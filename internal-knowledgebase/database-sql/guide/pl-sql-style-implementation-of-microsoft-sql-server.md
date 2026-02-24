---
tool_name: internal-knowledgebase
doc_type: guide
category: database-sql
title: "PL/SQL-style implementation of Microsoft SQL Server date functions"
summary: "Build compatibility UDFs for DATEADD, DATEPART, DATEDIFF, and DATENAME during SQL Server to Exasol migration."
---
# PL/SQL-style implementation of Microsoft SQL Server date functions

## Purpose

When migrating workloads from Microsoft SQL Server, you may need compatibility wrappers for date/time functions frequently used in legacy SQL:

- `DATEADD`
- `DATEPART`
- `DATEDIFF`
- `DATENAME`

This guide provides a reusable Exasol UDF approach.

## Scope and caveats

- Function semantics are best-effort compatibility and may not match SQL Server behavior for every edge case.
- Validate behavior for locale-sensitive names, week calculations, and end-of-month handling.
- Keep implementation in a dedicated schema to avoid naming conflicts.

## Step 1: Create utility schema

```sql
CREATE SCHEMA IF NOT EXISTS UTILS;
```

## Step 2: Create compatibility functions

Create functions in `UTILS` schema:

- `UTILS.DATEADD(interval, interval_value, date_expr)`
- `UTILS.DATEPART(part_expr, date_expr)`
- `UTILS.DATEDIFF(datepart, start_date_expr, end_date_expr)`
- `UTILS.DATENAME(part_expr, date_expr)`

Optional helper functions:

- `UTILS.GET_FORMAT_FROM_STYLE(style)`
- `UTILS.CONVERT_TO_DATE(expr, style)`

Note: Use the internal migration script version of these function bodies where available.

## Step 3: Validate behavior

Run basic checks after deployment:

```sql
SELECT UTILS.DATEDIFF('DAY', TIMESTAMP '2024-01-01 00:00:00', TIMESTAMP '2024-01-03 00:00:00');
SELECT UTILS.DATEADD('MONTH', 1, TIMESTAMP '2024-01-31 12:00:00');
SELECT UTILS.DATEPART('WEEK', TIMESTAMP '2024-02-15 10:20:30');
SELECT UTILS.DATENAME('MONTH', TIMESTAMP '2024-02-15 10:20:30');
```

## Step 4: Migrate SQL usage

Replace SQL Server function calls in migrated SQL with `UTILS.*` calls, or create a translation layer in your migration pipeline.

## Recommended regression checks

- Month-end arithmetic (`Jan 31 + 1 month`, leap years).
- Week and weekday interpretation.
- Millisecond/microsecond precision handling.
- Locale-dependent month/day names.

## Reference

- <https://docs.exasol.com/sql/create_function.htm>


