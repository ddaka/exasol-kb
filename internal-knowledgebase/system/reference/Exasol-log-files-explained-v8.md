---
tool_name: confd_client
doc_type: reference
category: system
title: "Exasol log files explained v8"
summary: "There are certain log files for different services within Exasol. Logs are located in different locations. As log files will grow, there is daily log rotation (not necessary..."
---
# Exasol log files explained v8

## Overview

There are certain log files for different services within Exasol. Logs are located in different locations.
As log files will grow, there is daily log rotation (not necessary around 04:00 AM). At this time, every log will be packed, compressed and moved to `.logbackup` directory within each subfolder.

## Explanation

### Log file locations

### Exasol logs

In version 8 all Exasol related logs are relocated to
`/exa/logs/`

Outside of the COS, directly on the host itself, you can find the logs also in `/.ccc/play/local/<ID>/main/<NODE_ID>/data/logs/`.

### OS-Logs

`/var/log/`

### Details logs

### EXAClusterOS processes

The following logs are found in
`/exa/logs/logd/*`

`Authentication.log` - Authentication events
`DWAD.log` & `EXASolution_<dababase_name>`- Database events
`ConfD.log` - ConfD logs
`Lockd.log`
`Load.log` - Load on cluster nodes
`Storage.log` - Storage events

For every log you can also execute `logd_collect <Service>`
For example for all storage events: `logd_collect Storage`
For all database events: `logd_collect EXASolution_<dababase_name>`

### Database logs

The logs for the database itself are located in `/exa/logs/db/<database_name>/`.
There are different kind of logs, we differentiate between **SQL processes** and **Server processes**.
**SQL processes** include:

- SqlProcess
- SqlSession
- EtlProcess
- EtlJdbc

**Server processes** include:

- PddServer
- ObjectServer
- SqLogServer
- LoaderD
- ConnectionServer
- controller

These log files always have a number at the end. This number is the logical number of the node. You can get this number with `dwad_client sys-nodes <database_name>`.
For example:

````text
Test [root@n0010 ~]# dwad_client sys-nodes Test_DB
                 NODENAME           STATE      IPROC  IPROC (CONTR.)    PDD NR.
n0011.c0001.exacluster.local         running          0               0          0
n0012.c0001.exacluster.local         running          1               1          1
n0013.c0001.exacluster.local         reserve          -               -          -
````

Here the node n11 has the logical number 0.

### Coredumps

Coredumps can be found on every data nodes in `/exa/spool/coredumps/`

### Collecting log files

Since there is no EXAoperation anymore you have to collect the logs via command line.

See also [Log Files for Support](https://docs.exasol.com/db/latest/administration/on-premise/support.htm) for details.

**command line**
With `exasupport` you can collect log files from within the cluster. The logs are by default saved in `/exa/tmp/support/`.
Outside of the COS you can find them in `/.ccc/play/local/<ID>/main/<NODE_ID>/data/logs/tmp/support/`. Keep in mind that these files are owned by root.

Use `exasupport -h` to get the help for this tool.

```text
 exasupport -h
usage: exasupport [-h] [-d DEBUG_INFO] [-e EXASOLUTION] [-x EXASOLUTION_LOG_TYPE] [-b BACKTRACES] [-i SESSION] [-n NODES] [-a] [-m] [-o OUTFILE] [-s START_TIME]
                  [-t STOP_TIME]

Retrieve support information

options:
  -h, --help            show this help message and exit
  -d DEBUG_INFO, --debug-info DEBUG_INFO
                        Debug info to retrieve, separated by comma: 1 = EXAClusterOS logs, 2 = Coredumps, 3 = EXAStorage metadata, or 0 for all
  -e EXASOLUTION, --exasolution EXASOLUTION
                        EXASolution logs (System names, separated by comma or "All databases")
  -x EXASOLUTION_LOG_TYPE, --exasolution-log-type EXASOLUTION_LOG_TYPE
                        EXASolution log type (name or digit; one of: "1" or "All", "2" or "SQL processes", "3" or "Server processes")
  -b BACKTRACES, --backtraces BACKTRACES
                        Process backtraces 1 = EXASolution server processes, 2 = EXASolution SQL processes, 3 = EXAClusterOS processes, 4 = ETL JDBC Jobs
  -i SESSION, --session SESSION
                        Collect logs from this session (can be specified more than once)
  -n NODES, --nodes NODES
                        Nodes, separated by comma (default: all online nodes)
  -a, --only-archives   Only download archives, i.e. rotated logs
  -m, --estimate        Only estimate size of debug information
  -o OUTFILE, --outfile OUTFILE
                        Output file to local, or remote:<remote volume name>,<path in volume>
  -s START_TIME, --start-time START_TIME
                        Start time of logs (YYYY-MM-DD[ HH:MM])
  -t STOP_TIME, --stop-time STOP_TIME
                        Stop time of logs (YYYY-MM-DD[ HH:MM])
```

The options `-e` for databases and `-x` for EXASolution logs are mandatory.

SQL processes logs are the largest available logs. Therefore, please pull them only when necessary. Information on which logs are needed in which situation could be found in [Support scenarios and needed resources](https://exasol.atlassian.net/wiki/spaces/SPOTSUP/pages/4555971/Support+scenarios+and+needed+resources).
