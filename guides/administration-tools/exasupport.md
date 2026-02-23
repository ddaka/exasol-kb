---
tool_name: confd_client
doc_type: reference
category: administration
subcommands:
  - exasupport
technical_entities:
  - EXAsupport
  - logs
  - debug information
  - coredumps
  - backtraces
  - support archive
  - c4 connect
summary: >
  EXAsupport command-line tool reference — how to access via c4 connect, full
  command options (debug-info, start/stop time, log types, sessions, backtraces,
  nodes, estimate, output file), and checking free disk space before collection.
---

# EXAsupport

This article explains how to access and use the EXAsupport tool in Exasol.

If you have an issue with your Exasol system and need help, create a support case. To be able to troubleshoot and resolve your issue, our Support team will require logs and other information about your system. This section explains how to use EXAsupport to retrieve the required information.

In Exasol 2025.1 and later you can use **Exasol Admin** to automatically create an archive with the necessary logs, which you can then include with your support case. For earlier versions of Exasol, you must retrieve the logs manually using EXAsupport.

---

## Access EXAsupport

EXAsupport is available on all nodes in an Exasol deployment. To access EXAsupport you must first connect to the cluster operating system (COS) in the deployment using c4.

To connect to COS, use `c4 connect -t <DEPLOYMENT>[.<NODE>]/cos`. For example:

```bash
./c4 connect -t 1/cos
```

Once you are connected to COS, use `exasupport -h` to view the EXAsupport help.

The user executing the EXAsupport commands is the user who is currently logged into the node, which is typically `root`. There are no extra authentication steps when using EXAsupport in this way.

---

## Command Options

The following list describes all the options for the `exasupport` command and the default behavior if an option is omitted.

- **`--help`, `-h`** — Lists the options, definitions, and syntax of the `exasupport` command.

- **`--debug-info=<DEBUGINFO>`, `-d <DEBUGINFO>`** — Comma-separated list of the types of debug information to collect:
  - `EXAClusterOS logs` or `1`
  - `Coredumps` or `2`
  - `EXAStorage metadata` or `3`
  - `0` = collect all debug information
  - Default: No information is collected.

- **`--start-time=<YYYY-MM-DD [HH:MM]>`, `-s`** — Logs older than this time will not be collected. Default: Logs of any age are collected.

- **`--stop-time=<YYYY-MM-DD [HH:MM]>`, `-t`** — Logs more recent than this time will not be collected. Default: Logs up to the present time are collected.

- **`--exasolution=<DATABASES>`, `-e`** — Comma-separated list of Exasol systems to collect logs from. Default: All databases.

- **`--exasolution-log-type=<TYPE>`, `-x`** — The type of logs to collect:
  - `All` or `1`
  - `SQL processes` or `2`
  - `Server processes` or `3`
  - Default: No logs are collected, except logs from sessions defined in `--session`.

- **`--session=<SESSION>`, `-i`** — The sessions from which to collect logs. Multiple sessions are defined by repeating the option. Default: No additional session logs are collected.

- **`--backtraces=<BACKTRACES>`, `-b`** — Defines process backtraces to collect:
  - `EXASolution server processes` or `1`
  - `EXASolution SQL processes` or `2`
  - `EXAClusterOS processes` or `3`
  - `ETL JDBC Jobs` or `4`
  - Default: Backtraces are not collected.

- **`--nodes=<NODES>`, `-n`** — Comma-separated list of node IDs to include. Default: All online nodes.

- **`--only-archives`, `-a`** — Only rotated logs are downloaded. Default: Both rotated and non-rotated logs.

- **`--estimate`, `-m`** — Calculate uncompressed size of debug information without collecting (dry run). Default: Logs are collected.

- **`--outfile=<OUTFILE>`, `-o`** — Path to a local file or remote volume for output. Format: `[<remote volume name>,]<path in volume>`. Default: `/exa/tmp/support/exacluster_debuginfo_[TIMESTAMP].tar.gz`

### Example

This command retrieves all SQL logs from database MY_DATABASE from 2024-06-15 to present:

```bash
exasupport -e MY_DATABASE -x 2 -s 2024-06-15 -o ./my_log.tar.gz
```

---

## Check Free Disk Space

Logs can become very large. Before you generate a log, determine the estimated size and make sure you have enough free disk space.

1. To estimate the size, add `-m` at the end of the EXAsupport command:

```bash
exasupport -x 2 -m
# Output: 12.34 MiB
```

2. To check available free disk space on the node:

```bash
df -h
```
