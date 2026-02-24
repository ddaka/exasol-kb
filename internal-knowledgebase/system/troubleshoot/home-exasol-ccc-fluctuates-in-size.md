---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "/home/exasol/.ccc fluctuates in size"
summary: "The customer reports that the disk usage of `/home/exasol/.ccc` fluctuates uncontrollably between 50GB and 90GB on a daily basis."
---
# /home/exasol/.ccc fluctuates in size

## Problem (SF case: 00080417)

The customer reports that the disk usage of `/home/exasol/.ccc` fluctuates uncontrollably between 50GB and 90GB on a daily basis.

The customer has a script which checks the disk usage of the folder and they shared the output with us:

```bash
root@ldexanode02:~# cat file_list.txt | grep -E "Auflistung|.ccc"
Auflistung vom 2025-01-06 15:08:18
40G     .ccc
Auflistung vom 2025-01-06 15:09:06
41G     .ccc
Auflistung vom 2025-01-06 15:30:01
56G     .ccc
Auflistung vom 2025-01-06 16:00:01
56G     .ccc
Auflistung vom 2025-01-06 16:30:01
56G     .ccc
Auflistung vom 2025-01-06 17:00:01
56G     .ccc
Auflistung vom 2025-01-06 17:30:01
56G     .ccc
Auflistung vom 2025-01-06 18:00:01
56G     .ccc
Auflistung vom 2025-01-06 18:30:01
56G     .ccc
Auflistung vom 2025-01-06 19:00:01
67G     .ccc
Auflistung vom 2025-01-06 19:30:01
67G     .ccc
Auflistung vom 2025-01-06 20:00:01
67G     .ccc
Auflistung vom 2025-01-06 20:30:01
67G     .ccc
Auflistung vom 2025-01-06 21:00:01
67G     .ccc
Auflistung vom 2025-01-06 21:30:01
67G     .ccc
Auflistung vom 2025-01-06 22:00:01
67G     .ccc
Auflistung vom 2025-01-06 22:30:01
73G     .ccc
Auflistung vom 2025-01-06 23:00:01
73G     .ccc
...
Auflistung vom 2025-01-07 03:00:01
73G     .ccc
Auflistung vom 2025-01-07 03:30:01
31G     .ccc
Auflistung vom 2025-01-07 04:00:02
31G     .ccc
....
Auflistung vom 2025-01-07 09:30:01
36G     .ccc
Auflistung vom 2025-01-07 10:00:02
36G     .ccc
Auflistung vom 2025-01-07 10:30:01
36G     .ccc
Auflistung vom 2025-01-07 11:00:01
36G     .ccc
Auflistung vom 2025-01-07 11:30:01
36G     .ccc
Auflistung vom 2025-01-07 12:00:01
36G     .ccc
Auflistung vom 2025-01-07 12:30:01
36G     .ccc
```
The disk usage decreases once the daily log rotation (`/etc/cron.daily/exa-logrotate`) runs, this indicates that some DB logs are flooded by wasteful entries.

It was discovered that the log of IMPORT from PostgreSQL is flooded with such entries:
```
Jan 06, 2025 3:15:35 PM org.postgresql.util.LazyCleaner$1 run
WARNING: Unexpected exception in cleaner thread main loop
java.security.AccessControlException: access denied ("java.lang.RuntimePermission" "setContextClassLoader")
at java.base/java.security.AccessControlContext.checkPermission(AccessControlContext.java:472)
at java.base/java.security.AccessController.checkPermission(AccessController.java:895)
at java.base/java.lang.SecurityManager.checkPermission(SecurityManager.java:322)
at java.base/java.lang.Thread.setContextClassLoader(Thread.java:1522)
at org.postgresql.util.LazyCleaner$1.run(LazyCleaner.java:126)
at java.base/java.lang.Thread.run(Thread.java:834)
```

## Procedure

Alter the PostgreSQL JDBC driver configuration file (settings.cfg) by adding the line ->  `"NOSECURITY=YES"` and re-upload the file to the same location as the driver.

## Additional References

[Manage Buckets and Files in BucketFS](https://docs.exasol.com/db/latest/administration/on-premise/bucketfs/file_access.htm)

[Add JDBC Driver](https://docs.exasol.com/db/latest/administration/on-premise/manage_drivers/add_jdbc_driver.htm)
