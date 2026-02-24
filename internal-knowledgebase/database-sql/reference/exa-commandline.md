---
tool_name: internal-knowledgebase
doc_type: reference
category: database-sql
title: "EXA_COMMANDLINE System Table"
summary: "Reference for inspecting current database command-line parameter values using EXA_COMMANDLINE (internal, non-historical view)."
---

# EXA_COMMANDLINE System Table

## Overview

`EXA_COMMANDLINE` exposes current command-line parameter values known to the database instance.

Important:

- This table is internal and not publicly documented.
- It reflects current state and is not a historical audit table.

## What You Can Inspect

- Parameter names
- Current effective values
- Static/dynamic marker (`IS_STATIC`)

Example parameters visible in this table include replication border settings such as:

- `soft_replicationborder_in_kb`
- `soft_replicationborder_in_numrows`

## Example Query

```sql
SELECT *
FROM EXA_COMMANDLINE
ORDER BY PARAM_NAME;
```

You can also filter for relevant parameters:

```sql
SELECT param_name, param_value, is_static
FROM EXA_COMMANDLINE
WHERE LOWER(param_name) LIKE '%replicationborder%'
ORDER BY param_name;
```

## Notes

- `IS_STATIC = FALSE` indicates runtime mutability in principle, but runtime changes are generally not supported operationally.
- Treat values in this table as runtime diagnostics, not as change history.
