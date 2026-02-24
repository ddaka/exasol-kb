---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: database-sql
title: "No DB Stats for 30 Minutes"
summary: "Investigate missing database statistics alerts and recover session-stat collection when stuck check_sql processes are present."
---
# No DB Stats for 30 Minutes

## Problem

An alert is triggered with details similar to:

**Case Subject:** No DB Stats for 30 Minutes
**Case Description:** Critical case. No database statistics have been received for the past 30 minutes.

## Procedure

1. Check the current database status.
1. If the database restarted recently, inform the customer and begin RCA.
1. If the database is stopped, start it and continue RCA.
1. If the database is online and queryable (for example with `exa_debug`), continue with process checks.
1. Check for timed-out `check_sql` processes on nodes:

```shell
psh "ps -ef | grep -i check_sql"
```

1. If a stale `check_sql` process is confirmed, connect to the affected node and terminate only the identified stuck PID:

```shell
ssh <node-name>
kill -9 <check_sql_process_id>
```

1. Verify that statistics/session activity resumes in Grafana:

`Grafana Home` -> `Database Statistics` -> `clientID` -> `clusterID` -> `database` -> `All Sessions`

## Result

The alert is resolved when database statistics are reported again and monitoring returns to normal.

---

_We welcome feedback on this troubleshooting article._
