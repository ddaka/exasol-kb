---
tool_name: internal-knowledgebase
doc_type: reference
category: system
title: "3rd Level statistics for performance analysis"
summary: "In order to analyse the database performance R&D needs to have a deep look into the EXA_STATISTICS. These scripts are very helpful e.g. if the size of the system has changed or if..."
---
# 3rd Level statistics for performance analysis

### Overview

In order to analyse the database performance R&D needs to have a deep look into the EXA_STATISTICS. These scripts are very helpful e.g. if the size of the system has changed or if the performance is not as expected. Below scripts create CSVs of all necessary tables. Please run those scripts on the **mgmt. node** or **outside the cluster**. Please do not run them on database nodes as those scripts can create GiBs of CSV data. Furthermore, please keep in mind that those **scripts require a read-access to the EXA_STATISTICS** table **AND** an **active session slot**. For already struggling systems please always inform the customer about creating those statistics. We recommend to create a so-called ***debug*** user to create stats. The statistics do not contain any user data it is plain performance statistics.

### Debug User

```bash
CREATE USER exa_debug IDENTIFIED BY "secure password";
GRANT CREATE SESSION TO exa_debug;
GRANT SELECT ANY DICTIONARY TO exa_debug;
GRANT EXPORT TO exa_debug;
```
### RdLevelStatsHourly

Requires the number of days to pull '**define days**'. It can be set on the fly - exaplus will ask for input or inside the SQL or as a commandline parameter for exaplus.

```bash
exaplus -c <<connection_string>> -u <<user>> -P <<pwd>> -f 3rdLevelStatsHourly.sql -- <<numberOfDays>>
```

```bash
set autocommit off;

define days = &1;

alter session set NLS_DATE_FORMAT = 'YYYY-MM-DD';
alter session set NLS_FIRST_DAY_OF_WEEK = 7;
alter session set NLS_TIMESTAMP_FORMAT = 'YYYY-MM-DD HH:MI:SS.FF6';
alter session set NLS_NUMERIC_CHARACTERS = '.,';
alter session set NLS_DATE_LANGUAGE = 'ENG';
alter session set QUERY_TIMEOUT = 0;
alter session set SQL_PREPROCESSOR_SCRIPT = '';

export (select * from "$EXA_MONITOR_HOURLY" where INTERVAL_START > ADD_DAYS(systimestamp, -&days)) into local csv file 'monitor_hourly.csv' truncate;
export (select * from EXA_USAGE_HOURLY where INTERVAL_START > ADD_DAYS(systimestamp, -&days)) into local csv file 'usage_hourly.csv' truncate;
export (select * from "$EXA_SQL_HOURLY" where INTERVAL_START > ADD_DAYS(systimestamp, -&days)) into local csv file 'sql_hourly.csv' truncate;
export (select * from "$EXA_PROFILE_HOURLY" where INTERVAL_START > ADD_DAYS(systimestamp, -&days)) into local csv file 'profile_hourly.csv' truncate;
export (select * from "$EXA_DB_SIZE_HOURLY" where INTERVAL_START > ADD_DAYS(systimestamp, -&days)) into local csv file 'db_size_hourly.csv' truncate;
export (select * from "$EXA_SYSTEM_EVENTS" where MEASURE_TIME >= (SELECT LEAST(ADD_DAYS(systimestamp, -&days), MAX(MEASURE_TIME)) FROM "$EXA_SYSTEM_EVENTS" WHERE EVENT_TYPE='STARTUP')) into local csv file 'system_events.csv' truncate;
export (select param_value from exa_metadata where param_name = 'databaseProductVersion') into local csv file 'database_version.csv' truncate;

select 'exported hourly statistics of last &days days to monitor_hourly.csv, usage_hourly.csv, sql_hourly.csv, profile_hourly.csv, db_size_hourly.csv, system_events.csv, database_version.csv';
```
### RdLevelStatsIndices

These stats can take quite long - talking about 30 minutes and more especially if the system is under high load.

```bash
set autocommit off;

alter session set NLS_DATE_FORMAT = 'YYYY-MM-DD';
alter session set NLS_FIRST_DAY_OF_WEEK = 7;
alter session set NLS_TIMESTAMP_FORMAT = 'YYYY-MM-DD HH:MI:SS.FF6';
alter session set NLS_NUMERIC_CHARACTERS = '.,';
alter session set NLS_DATE_LANGUAGE = 'ENG';
alter session set QUERY_TIMEOUT = 0;
alter session set SQL_PREPROCESSOR_SCRIPT = '';

export (select * from "EXA_VOLUME_USAGE") into local csv file 'exa_volume_usage.csv' truncate;
export (select * from "$EXA_INDICES") into local csv file 'exa_indices.csv' truncate;
export (select * from "$EXA_COLUMN_SIZES") into local csv file 'exa_column_sizes.csv' truncate;
export (select * from "$EXA_COLUMN_STATISTICS") into local csv file 'exa_column_statistics.csv' truncate;

select 'exported object statistics to exa_volume_usage.csv, exa_indices.csv, exa_column_sizes.csv, exa_column_statistics.csv';
```
### RdLevelStatsLastDay

```bash
set autocommit off;

alter session set NLS_DATE_FORMAT = 'YYYY-MM-DD';
alter session set NLS_FIRST_DAY_OF_WEEK = 7;
alter session set NLS_TIMESTAMP_FORMAT = 'YYYY-MM-DD HH:MI:SS.FF6';
alter session set NLS_NUMERIC_CHARACTERS = '.,';
alter session set NLS_DATE_LANGUAGE = 'ENG';
alter session set QUERY_TIMEOUT = 0;
alter session set SQL_PREPROCESSOR_SCRIPT = '';

export (select * from "$EXA_MONITOR_DETAILS_LAST_DAY") into local csv file 'monitor_details_last_day.csv' truncate;
export (select * from EXA_USAGE_LAST_DAY) into local csv file 'usage_last_day.csv' truncate;
export (select * from "$EXA_SQL_LAST_DAY") into local csv file 'sql_last_day.csv' truncate;
export (select * from "$EXA_PROFILE_LAST_DAY") into local csv file 'profile_last_day.csv' truncate;
export (select * from "$EXA_DB_SIZE_LAST_DAY") into local csv file 'db_size_last_day.csv' truncate;
export (select SESSION_ID,LOGIN_TIME,LOGOUT_TIME,CLIENT,DRIVER,ENCRYPTED,SUCCESS,ERROR_CODE,ERROR_TEXT from EXA_DBA_SESSIONS_LAST_DAY) into local csv file 'all_sessions.csv' truncate;
export (select * from "$EXA_SYSTEM_EVENTS" where MEASURE_TIME >= (SELECT LEAST(ADD_DAYS(systimestamp, -30), MAX(MEASURE_TIME)) FROM "$EXA_SYSTEM_EVENTS" WHERE EVENT_TYPE='STARTUP')) into local csv file 'system_events_30days.csv' truncate;
export (select * from EXA_DBA_TRANSACTION_CONFLICTS where coalesce(STOP_TIME,START_TIME) > add_hours(systimestamp, -24)) into local csv file 'transaction_conflicts.csv' truncate;
export (select param_value from exa_metadata where param_name = 'databaseProductVersion') into local csv file 'database_version.csv' truncate;

select 'exported last day statistics to monitor_details_last_day.csv, usage_last_day.csv, sql_last_day.csv, profile_last_day.csv, db_size_last_day.csv, all_sessions.csv, system_events_30days.csv, transaction_conflicts.csv, database_version.csv';
```

## Downloads

[3rd_level_SQL_files.zip](https://github.com/exasol/Internal-Knowledgebase/files/9987035/3rd_level_SQL_files.zip)
