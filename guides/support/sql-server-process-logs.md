---
tool_name: confd_client
doc_type: guide
category: support
subcommands:
  - db_list
technical_entities:
  - EXAsupport
  - SQL and server process logs
  - transaction conflicts
summary: >
  How to collect combined SQL and server process logs using EXAsupport — for
  transaction conflicts, using exasupport -x 1.
---

# Logs for SQL and Server Processes

If you open a support case for transaction conflicts, you need to provide logs
for the SQL and server processes for the time period when the problem occurred.

In Exasol **2025.1+** you can alternatively use Exasol Admin to create a support
archive.

## Prerequisites

Enough free disk space for the logs.

## Procedure

1. Connect to COS:

```bash
c4 connect -i c3275f84 -s cos
```

2. Find the database name:

```bash
confd_client db_list
```

3. Collect SQL and server process logs:

```bash
exasupport -x 1 -s 2022-08-11 -t 2022-08-11 -e MY_DATABASE
```

| Flag   | Description                                      |
|--------|--------------------------------------------------|
| `-x 1` | All log types (SQL + server processes combined)  |
| `-s`   | Start date (`YYYY-MM-DD`)                        |
| `-t`   | End date (`YYYY-MM-DD`)                          |
| `-e`   | Database name (omit for all databases)           |

4. Disconnect from COS (`Ctrl+D` or `exit`).

5. Copy the file to your local machine:

```bash
c4 connect -t 1.11/cos -- "cat /exa/tmp/support/$FILENAME" > $FILENAME
```

## Verification

```bash
ls -lahtr ~/mylogs | grep exacluster_debuginfo
```
