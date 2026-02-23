---
tool_name: confd_client
doc_type: guide
category: support
subcommands:
  - db_list
technical_entities:
  - EXAsupport
  - server process logs
summary: >
  How to collect server process logs using EXAsupport — for "system out of
  memory" errors and database engine issues, using exasupport -x 3.
---

# Logs for Server Processes

If you open a support case for "system out of memory" errors, you need to
provide log files for the server processes for the time period when the problem
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

4. Collect server process logs:

```bash
exasupport -x 3 -s 2022-08-11 -t 2022-08-11 -e MY_DATABASE
```

| Flag   | Description                                  |
|--------|----------------------------------------------|
| `-x 3` | Server process logs only                     |
| `-s`   | Start date (`YYYY-MM-DD`)                    |
| `-t`   | End date (`YYYY-MM-DD`)                      |
| `-e`   | Database name (omit for all databases)       |

5. Disconnect from COS (`Ctrl+D` or `exit`).

6. Copy the file to your local machine:

```bash
c4 connect -t 1.11/cos -- "cat /exa/tmp/support/$FILENAME" > $FILENAME
```

## Verification

```bash
ls -lahtr ~/mylogs | grep exacluster_debuginfo
```
