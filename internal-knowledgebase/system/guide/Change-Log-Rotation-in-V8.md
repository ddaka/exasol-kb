---
tool_name: cos
doc_type: guide
category: system
title: "Change log rotation schedule in v8"
summary: "Modify and propagate cron-based log rotation schedule in Exasol v8 COS."
---
# Change log rotation schedule in v8

## Purpose

Adjust the log rotation schedule in Exasol v8 by updating cron configuration in COS.

## Prerequisites

- COS shell access on cluster nodes.
- Change window and rollback plan.

## Procedure

### 1. Edit rotation schedule

Update cron definition in:

```text
/etc/cron.d/anacron
```

Example schedules:

```cron
30 7-23 * * * root cp -f /etc/file1 /etc/logfiles/
25 2 * * * root cp -f /etc/file1 /etc/log/
```

Use standard cron syntax and validate expression before rollout.

### 2. Sync file across nodes

```shell
cos_sync_files /etc/cron.d/anacron
```

### 3. Reload cron service

```shell
psh "service cron reload"
```

### 4. Verify rotation behavior

After expected execution window, verify rotated log timestamps:

```shell
ls -lthr /exa/logs/db/<database-name>/.logbackup
```

## Result

Updated log rotation schedule is active and consistent across nodes.


