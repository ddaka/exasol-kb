---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "How to find DRS log files"
summary: "Locate Data Replication Service (DRS) log files by inspecting running processes or DRS configuration paths on the DRS service host."
---
# How to find DRS log files

## Purpose

Locate log files for Exasol Data Replication Service (DRS), which runs as an external service with its own host and log location.

## Prerequisites

- SSH access to DRS service host.
- Basic shell access (`pgrep`, `ps`, `grep`, `lsof`).
- Optional: DB access to identify DRS host via session metadata.

## 1) Identify DRS service host

If service location is undocumented, inspect DRS-related DB sessions (`EXA_DBA_SESSIONS` / `EXA_DBA_AUDIT_SESSIONS`) and use `HOST` to identify the DRS machine.

## 2) Locate logs from running process (preferred)

Find DRS PID:

```shell
pgrep --list-full --full "data-replication"
# or
ps ax | grep "data-replication"
```

Inspect open file handles:

```shell
ls -l /proc/<pid>/fd/
# or
lsof -p <pid> | grep 'w *REG'
```

This reveals the active log file path (for example under `/usd/.../work/drs-logs/`).

## 3) Locate logs from configuration (if process is down)

Check DRS config folders:

- `/etc/exasol-drs`
- `${HOME}/.config/exasol-drs`

Read logger target from `config.ini`:

```shell
grep '^logger=' config.ini
```

Example:

```text
logger=file\:///usd/as16561a/work/drs-logs,exasol-drs
```

This indicates log directory and file prefix.

## Environment-specific shortcut (Finanz Informatik)

For FI environments, common path pattern is:

- `/usd/${Dienstname}a/work/drs-logs/`

## COS correlation logs (validated against COS docs)

For platform-side correlation during incidents, collect COS service logs with `confd_client log_collect`:

```shell
confd_client log_collect {services: [ConfD, DWAd, HealthD], start_time: 'YYYY-MM-DD HH:MM:SS', stop_time: 'YYYY-MM-DD HH:MM:SS'}
```

## Notes

- DRS may use non-file loggers; in that case no local log files may exist.
- DRS 2.1.1+ can set `DRS_CONFIG_FOLDER` via environment variable in management scripts; if process is down and folder is undocumented, full filesystem search may be required.


