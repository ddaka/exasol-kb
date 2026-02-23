---
tool_name: confd_client
doc_type: guide
category: support
subcommands:
  - db_list
technical_entities:
  - EXAsupport
  - logs
  - backtraces
  - coredumps
summary: >
  How to collect all logs, backtraces, and system information in a single
  EXAsupport command — complete procedure with c4/COS connection, exasupport
  flags, and file copy to local machine.
---

# Get All Logs

Collect all logs and information for a support case in a single operation.

In Exasol **2025.1+** you can alternatively use Exasol Admin to create a support
archive. For earlier versions, use EXAsupport.

## Prerequisites

Enough free disk space for the logs. Use `exasupport -m` to estimate.

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

4. Collect all logs with EXAsupport:

```bash
exasupport -d 0 -x 1 -b 1,2,3,4 -s 2022-08-11 -t 2022-08-11 -e MY_DATABASE
```

| Flag   | Description                                              |
|--------|----------------------------------------------------------|
| `-d 0` | Debug info: all (COS logs, coredumps, EXAStorage metadata) |
| `-x 1` | All log types (SQL + server processes)                   |
| `-b 1,2,3,4` | All backtraces (server, SQL, COS, ETL JDBC)       |
| `-s`   | Start date (`YYYY-MM-DD`)                                |
| `-t`   | End date (`YYYY-MM-DD`)                                  |
| `-e`   | Database name (omit for all databases)                   |

5. Disconnect from COS (`Ctrl+D` or `exit`).

6. Copy the file to your local machine:

```bash
c4 connect -t 1.11/cos -- "cat /exa/tmp/support/$FILENAME" > $FILENAME
```

## Verification

```bash
ls -lahtr ~/mylogs | grep exacluster_debuginfo
```
