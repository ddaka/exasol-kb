---
tool_name: confd_client
doc_type: guide
category: support
subcommands:
  - db_list
technical_entities:
  - EXAsupport
  - profiling
  - EXA_USER_PROFILE_LAST_DAY
  - session
summary: >
  How to collect profiling information for a problematic SQL query — enable
  profiling, export profile output CSV, then collect session logs using
  EXAsupport -i SESSION_ID.
---

# Profiling Information

In some scenarios Support may ask you to provide profiling information for a
problematic query. To get this information you must first run a test of the
problematic statement with profiling enabled, then collect the log files for that
session using EXAsupport.

## Prerequisites

Enough free disk space for the logs.

## Step 1: Run a Test with Profiling Enabled

1. Open a new database connection using your preferred SQL client.
2. Execute the following (insert your problematic query):

```sql
ALTER SESSION SET PROFILE = 'on';

-- <insert your query here>

ALTER SESSION SET PROFILE = 'off';
ALTER SESSION SET NLS_NUMERIC_CHARACTERS = '.,';
ALTER SESSION SET NLS_TIMESTAMP_FORMAT = 'YYYY-MM-DD HH:MI:SS.ff3';
COMMIT;
FLUSH STATISTICS;
COMMIT;

EXPORT (
  SELECT *
  FROM EXA_STATISTICS.EXA_USER_PROFILE_LAST_DAY
  WHERE session_id = CURRENT_SESSION
)
INTO LOCAL CSV FILE 'profile_output.csv';
```

3. Attach the generated `profile_output.csv` to your support ticket.

## Step 2: Collect Log Files for the Test Session

1. Get the play ID:

```bash
c4 ps
```

2. Connect to COS:

```bash
c4 connect -i c3275f84 -s cos
```

3. Find the database name:

```bash
confd_client db_list
```

4. Collect logs for the test session:

```bash
exasupport -i SESSION_ID -s 2022-08-11 -t 2022-08-11 -e MY_DATABASE
```

> In Exasol versions prior to 8.25.0, you can only collect SQL logs for a
> specific session on the day the session occurred. Later versions do not have
> this limitation.

5. Disconnect from COS (`Ctrl+D` or `exit`).

6. Copy the file to your local machine:

```bash
c4 connect -t 1.11/cos -- "cat /exa/tmp/support/$FILENAME" > $FILENAME
```

## Verification

```bash
ls -lahtr ~/mylogs | grep exacluster_debuginfo
```
