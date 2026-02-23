---
tool_name: confd_client
doc_type: guide
category: support
subcommands:
  - db_list
technical_entities:
  - EXAsupport
  - cluster logs
  - COS
summary: >
  How to collect cluster (COS) logs using EXAsupport — for general cluster
  operating system issues, using exasupport -d1.
---

# Cluster Logs

If you open a support case for general issues with the cluster operating system
(COS), you need to provide a cluster log.

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

4. Collect cluster logs:

```bash
exasupport -d1 -s 2022-08-11 -t 2022-08-11 -e MY_DATABASE -x3
```

| Flag   | Description                                  |
|--------|----------------------------------------------|
| `-d1`  | COS logs                                     |
| `-x3`  | Server process logs (combined with COS)      |
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
