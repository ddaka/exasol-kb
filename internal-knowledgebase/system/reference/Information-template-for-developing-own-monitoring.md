---
tool_name: confd_client
doc_type: reference
category: system
title: "Reference for customer-developed monitoring in Exasol 8"
summary: "Data-source reference for customers building their own Exasol 8 monitoring, aligned with Exasol Cloud Monitoring concepts."
---
# Reference for customer-developed monitoring in Exasol 8

## Purpose

Provide a reference set of database tables and host/COS commands customers can use when implementing their own monitoring.

## 1) Statistical system tables

Commonly relevant statistical views:

- `EXA_DB_SIZE_HOURLY`
- `EXA_DB_SIZE_LAST_DAY`
- `EXA_MONITOR_DAILY`
- `EXA_MONITOR_HOURLY`
- `EXA_MONITOR_LAST_DAY`
- `EXA_SYSTEM_EVENTS`
- `EXA_USAGE_HOURLY`
- `EXA_USAGE_LAST_DAY`

Reference:

- <https://docs.exasol.com/db/latest/sql_references/system_tables/statistical_system_tables.htm>

## 2) Metadata system tables

Commonly relevant metadata views:

- `EXA_CLUSTERS`
- `EXA_DB_SNAPSHOTS`
- `EXA_LOADAVG`
- `EXA_METADATA`
- `EXA_OBJECTSTORAGE_USAGE`
- `EXA_VOLUME_USAGE`

Reference:

- <https://docs.exasol.com/db/latest/sql_references/system_tables/metadata_system_tables.htm>

## 3) Host/COS operational signals

Equivalent documented commands for key operational states:

- Network interfaces: `ip link` (host OS)
- Memory/huge pages: `cat /proc/meminfo` (host OS)
- Node info: `confd_client node_list`, `confd_client node_info`
- Volume info: `confd_client st_volume_list`, `confd_client st_volume_info`
- DB info/state: `confd_client db_info`, `confd_client db_state`
- Backup progress: `confd_client db_backup_progress`

## 4) Running COS commands from host

Example via `c4 connect`:

```shell
c4 connect -t 1/cos -n 11 -- 'confd_client node_list'
```

## Notes and limitations

- Internal Exasol monitoring may use internal tooling not exposed for customer use.
- Avoid persistent custom changes/scripts inside COS containers; updates/reboots can overwrite them.
- Revalidate command output and semantics after version upgrades.

## Additional references

- <https://exasol.my.site.com/s/article/Exasol-Usage-Insights-Hub-FAQ?language=en_US>
- <https://exasol.my.site.com/s/article/Monitoring-of-an-Exasol-Database?language=en_US>
- <https://github.com/exasol/nagios-monitoring/tree/master/opt/exasol/monitoring>


