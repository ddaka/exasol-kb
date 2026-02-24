---
tool_name: internal-knowledgebase
doc_type: reference
category: database-sql
title: "Lua Runtime Version in Exasol"
summary: "Reference for checking Lua runtime version used by Exasol UDFs and understanding language-feature compatibility constraints."
---

# Lua Runtime Version in Exasol

## Overview

Exasol may use a Lua runtime version that differs from current upstream Lua releases. Some newer Lua language features may therefore be unavailable.

Example of potentially unavailable syntax in older Lua runtimes: `goto`.

## Check Runtime Version

Run this UDF/script snippet to return the runtime version string:

```sql
CREATE OR REPLACE LUA SCALAR SCRIPT lua_version()
RETURNS VARCHAR(100) AS
function run(ctx)
    return _VERSION
end
/

SELECT lua_version() FROM dual;
-- example output: Lua 5.1
```

## Notes

- Validate script compatibility against the returned version before rollout.
- Keep this check in troubleshooting playbooks for Lua UDF migration issues.
