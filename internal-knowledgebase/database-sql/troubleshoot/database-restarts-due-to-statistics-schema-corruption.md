---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: database-sql
title: "Database restarts due to statistics schema corruption"
summary: "Recovery procedure for frequent restarts caused by corruption in EXA_STATISTICS system tables."
---
# Database restarts due to statistics schema corruption

## Problem

The database restarts repeatedly (for example every minute) because a corrupted object in `EXA_STATISTICS` prevents stable startup.

This procedure is a last-resort recovery path to recreate `EXA_STATISTICS`.

## Safety Notes

- Perform this only when corruption is confirmed and standard recovery paths are exhausted.
- Capture evidence and involve development support as needed.
- Preserve recoverable statistics data before dropping the schema.

## Step 1: Attempt to preserve `EXA_STATISTICS` data

1. Start database in maintenance mode (to avoid starting `SqlLogServer`).
1. Identify and exclude corrupted table(s).
1. Generate `EXPORT` statements for tables in `EXA_STATISTICS`.

```sql
select
  'EXPORT "' || h.object_name || '" into local csv file ''<path on local machine>/' || replace(h.object_name, '$') || '.csv.gz'' with column names;' as sql_text
from exa_hidden_objects h
where h.schema_name = 'EXA_STATISTICS'
  and h.object_type = 'TABLE'
  -- and h.object_name <> '<corrupted_table_name>'
order by h.object_name;
```

1. Run generated exports.
1. Prepare matching `IMPORT` statements for post-recovery restore.

```sql
select
  'IMPORT INTO "' || h.object_name || '" from local csv file ''<path on local machine>/' || replace(h.object_name, '$') || '.csv.gz'' skip=1;' as sql_text
from exa_hidden_objects h
where h.schema_name = 'EXA_STATISTICS'
  and h.object_type = 'TABLE'
  -- and h.object_name <> '<corrupted_table_name>'
order by h.object_name;
```

## Step 2: Recreate `EXA_STATISTICS`

1. Check current DB parameters:

```shell
confd_client db_info db_name: MY_DATABASE -j | jq -r '.config.params'
```

1. Stop database:

```shell
confd_client db_stop db_name: MY_DATABASE
```

1. Add temporary parameters to force recreation:

```shell
confd_client db_configure db_name: MY_DATABASE params_add: '[-dropStatisticsSchema=1 -useWriteLocksOnDroppedObjects=0]'
```

1. Start database:

```shell
confd_client db_start db_name: MY_DATABASE
```

1. Verify schema recreation, for example via:

```sql
select * from exa_system_events;
```

1. Stop database again:

```shell
confd_client db_stop db_name: MY_DATABASE
```

1. Remove temporary parameters (mandatory):

```shell
confd_client db_configure db_name: MY_DATABASE params_delete: '[-dropStatisticsSchema=1 -useWriteLocksOnDroppedObjects=0]'
```

1. Confirm parameters are removed:

```shell
confd_client db_info db_name: MY_DATABASE -j | jq -r '.config.params'
```

1. Start database:

```shell
confd_client db_start db_name: MY_DATABASE
```

## Step 3: Restore preserved statistics (optional)

If export data was preserved, run previously generated `IMPORT` statements in maintenance mode to restore non-corrupted statistics tables.

## Result

The incident is resolved when the database starts stably and `EXA_STATISTICS` is functional.

## References

- <https://docs.exasol.com/db/latest/administration/on-premise/manage_database/add_db_parameters.htm>
- <https://docs.exasol.com/db/latest/administration/on-premise/manage_database/remove_db_parameters.htm>
- <https://docs.exasol.com/db/latest/sql_references/system_tables/statistical_system_tables.htm>

---

_We welcome feedback on this troubleshooting article._
