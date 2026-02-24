---
tool_name: cos
doc_type: troubleshoot
category: system
title: "Exasol log files explained v7"
summary: "There are certain log files for different services within Exasol. Logs are located in different locations. As log files will grow, there is daily log rotation at 04:00 AM. At this..."
---
# Exasol log files explained v7

## Overview

There are certain log files for different services within Exasol. Logs are located in different locations.
As log files will grow, there is daily log rotation at 04:00 AM. At this time, every log will be packed, compressed and moved to `.logbackup` directory within each subfolder.

In EXAoperation under Monitoring you can change the deletion of the log files.
The default values are:
- Coredumps: 7 days
- SQL/server logs: 60 days

## Explanation

### Log file locations

### Database logs

`/d02_data/<database_name>/log/process`

### OS-Logs

`/var/log/`

### COS-Logs

    /var/log/logd
    /var/log/cored

### EXAoperation logs

`$COS_DIRECTORY/var/exaoperation/log/`

### Details logs

### EXAClusterOS processes

The following logs are found in
`/var/log/logd/*`

`Appserver.log` - here you can check for EXAoperation issues
`Authentication.log` - Authentication events
`DWAD.log` & `EXASolution_<dababase_name>`- Database events
`EXAoperation.log` - EXAoperation events
`Lockd.log`
`Load.log` - Load on cluster nodes
`Storage.log` - Storage events

For every log you can also execute `logd_collect <Service>`
For example for all storage events: `logd_collect Storage`
For all database events: `logd_collect EXASolution_<dababase_name>`

    /var/log/cored/*
     - /var/log/cored/appserverd
     - /var/log/cored/lockd
     - /var/log/cored/dwad
     - /var/log/cored/logd
     - /var/log/cored/cored
     - /var/log/cored/io_statistics
     - /var/log/cored/cos_storage

### Database logs

The logs for the database itself are located in `/d02_data/<database_name>/log/process`.
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
````
Test [root@n0010 ~]# dwad_client sys-nodes Test_DB
                 NODENAME           STATE      IPROC  IPROC (CONTR.)    PDD NR.
n0011.c0001.exacluster.local         running          0               0          0
n0012.c0001.exacluster.local         running          1               1          1
n0013.c0001.exacluster.local         reserve          -               -          -
````
Here the node n11 has the logical number 0.

### Coredumps

Coredumps can be found on every data nodes in `/d02_data/coredumps/`

### Logs during & after boot

During the boot of a node you can access it via `rssh nX`.
Log disk initialization on "to install" nodes:
`/var/log/hddinit.log`

Log disk partitioning on "active" nodes:
`/var/log/hddmount.log`

After booting logs of the boot process are available in `/var/log/exaopnodestart/`:
- `<private_node_IP>_start.log  `
- `<private_node_IP>_boot_ramdisk.tar.gz`

The **boot_ramdisk.tar.gz** contains the log files which you can find while being connect with `rssh nX`.

### Other logs

EXAoperation logs with its underlying zope DB logs.
`$COS_DIRECTORY/var/exaoperation/log/access.log`
`$COS_DIRECTORY/var/exaoperation/log/output.log`
`$COS_DIRECTORY/var/exaoperation/log/zope.log`
`$COS_DIRECTORY/var/exaoperation/log/zope_out.log`

### Collecting log files

There are two ways for collecting log files, either via EXAoperation or command line. For specific Sessions logs from the previous days you need to collect all sessions. The -i option does not work.

**EXAoperation**
See [Log Files for Support](https://docs.exasol.com/db/7.1/administration/on-premise/support.htm) for details.

**command line**
With `get_support_info` you can collect log files from within the cluster. Please keep in mind that collecting logs for multiple days might create a large file. Therefore it is recommended to collect them on a partition where you have a lot of free space.

Use `get_support_info -h` to get the help for this tool.
```
Options:
  -h, --help            show this help message and exit
  -d DEBUGINFO, --debug-info=DEBUGINFO
                        Debuginfo to retrieve, separated by comma: 1 =
                        EXAClusterOS logs, 2 = Coredumps, 3 = Ramdisk logs, 4
                        = EXAStorage metadata or 0 for all
  -s START_DATE, --start-time=START_DATE
                        Start time of logs (YYYY-MM-DD [HH:MM])
  -t STOP_DATE, --stop-time=STOP_DATE
                        Stop time of logs (YYYY-MM-DD [HH:MM])
  -e EXASOLUTION, --exasolution=EXASOLUTION
                        EXASolution logs (System names, separated by comma or
                        "All databases")
  -x EXASOLUTION_LOG_TYPE, --exasolution-log-type=EXASOLUTION_LOG_TYPE
                        EXASolution log type, separated by comma (1 = All, 2 =
                        SQL processes, 3 = Server processes)
  -i SESSION, --session=SESSION
                        Get logs from specific sessions, separated by comma
  -b BACKTRACES, --backtraces=BACKTRACES
                        Process backtraces 1 = EXASolution server processes, 2
                        = EXASolution SQL processes, 3 = EXAClusterOS
                        processes, 4 = ETL JDBC Jobs
  -n NODES, --nodes=NODES
                        Nodes (default: all online nodes)
  -a, --only-archives   Only download archives
  -f, --only-open-files
                        Only download open files
  -m, --estimate        Only estimate size of debug information
  -o OUTFILE, --outfile=OUTFILE
                        Output file

```
The options are the same as in EXAoperation, the options `-e` for databases and `-x` for EXASolution logs are mandatory. The default value for `-x` is **1**, so if you do not specify it, it will collect Server **AND** Sql processes.

SQL processes logs are the largest available logs. Therefore, please pull them only when necessary. Information on which logs are needed in which situation could be found in [Support scenarios and needed resources](https://exasol.atlassian.net/wiki/spaces/SPOTSUP/pages/4555971/Support+scenarios+and+needed+resources).
