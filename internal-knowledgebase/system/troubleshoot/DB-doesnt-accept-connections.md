---
tool_name: confd_client
doc_type: troubleshoot
category: system
title: "DB doesn't accept connections, though the database state is runnning"
summary: "DB doesn't accept connection although DB is in running state:"
---
# DB doesn't accept connections, though the database state is runnning

## Problem

DB doesn't accept connection although DB is in running state:

```text
root@n11:~# confd_client db_start db_name: Exasol
OK
root@n11:~# confd_client db_state db_name: Exasol
running
```

Symptoms, the warning message in PDD logs:

```text
[2025-07-16 11:04:01.745289] WARN: [Exasol] Controller(0.0): System msgsize_max is lower than requested: Using the system limit for max size of ipc messages (65536) instead of  Utils::system_value<int>(131072) as requested
```

## Procedure

check the `kernel.msgmax` size value using this command on OS (in cos you see the same output):

```shell
cat /proc/sys/kernel/msgmax
65536
cat /proc/sys/kernel/msgmnb
65536
```

If the value is low (something like above "65536"), change the value to "1073741824" using this command on OS:

```shell
sudo sysctl -w kernel.msgmax=1073741824
```

to make the change permanent, change it in this file as well:

```shell
 vi /etc/sysctl.conf
```

 navigate to the parameter "iniCopyEditkernel.msgmax" and change the value to 1073741824

```text
iniCopyEditkernel.msgmax=1073741824
```

Then save the changes to the file.

and reload the service with the command:

```shell
sudo sysctl -p
```

And restart database again.
