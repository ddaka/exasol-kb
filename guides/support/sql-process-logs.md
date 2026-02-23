---
tool_name: confd_client
doc_type: guide
category: support
subcommands:
  - db_list
technical_entities:
  - EXAsupport
  - SQL process logs
  - session logs
summary: >
  How to collect SQL process logs using EXAsupport — for SQL statement issues,
  using exasupport -x 2 for all SQL processes and exasupport -i SESSION_ID for
  specific sessions.
---

# Logs for SQL Processes

If you open a support case for problems with a specific SQL statement, you need
to provide log files for the SQL processes for the time period when the problem
occurred.

In Exasol **2025.1+** you can alternatively use Exasol Admin to create a support
archive.

## Prerequisites

Enough free disk space for the logs.

## Procedure

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

4. Collect SQL process logs:

```bash
exasupport -x 2 -s 2022-08-11 -t 2022-08-11 -e MY_DATABASE
```

5. To collect logs for a **specific session**:

```bash
exasupport -i SESSION_ID -s 2022-08-11 -t 2022-08-11 -e MY_DATABASE
```

> In Exasol versions prior to 8.25.0, you can only collect session logs on the
> day the session occurred. Later versions do not have this limitation.

| Flag   | Description                                  |
|--------|----------------------------------------------|
| `-x 2` | SQL process logs                             |
| `-i`   | Specific session ID                          |
| `-s`   | Start date (`YYYY-MM-DD`)                    |
| `-t`   | End date (`YYYY-MM-DD`)                      |
| `-e`   | Database name (omit for all databases)       |

6. Disconnect from COS (`Ctrl+D` or `exit`).

7. Copy the file to your local machine:

```bash
c4 connect -t 1.11/cos -- "cat /exa/tmp/support/$FILENAME" > $FILENAME
```

## Verification

```bash
ls -lahtr ~/mylogs | grep exacluster_debuginfo
```
