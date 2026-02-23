---
tool_name: confd_client
doc_type: guide
category: support
technical_entities:
  - EXAsupport
  - backtrace
  - coredumps
summary: >
  How to collect backtrace information using EXAsupport — for performance
  problems or hanging operations, using exasupport -b with backtrace type
  values for server, SQL, COS, and ETL JDBC processes.
---

# Backtrace Information

If you open a support case for performance problems or operations that hang, you
need to provide backtrace information for the time period when the problem
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

3. Collect backtraces:

```bash
exasupport -b 1,2,3,4
```

### Backtrace Types

| Value | Description                    |
|-------|--------------------------------|
| `1`   | EXASolution server processes   |
| `2`   | EXASolution SQL processes      |
| `3`   | EXAClusterOS processes         |
| `4`   | ETL JDBC Jobs                  |

4. Disconnect from COS (`Ctrl+D` or `exit`).

5. Copy the file to your local machine:

```bash
c4 connect -t 1.11/cos -- "cat /exa/tmp/support/$FILENAME" > $FILENAME
```

## Verification

```bash
ls -lahtr ~/mylogs | grep exacluster_debuginfo
```
