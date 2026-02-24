---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "Reduce kernel call-trace warning floods in LogService"
summary: "Operational steps to stop repeated kernel call-trace warning events from flooding EXAoperation LogService."
---
# Reduce kernel call-trace warning floods in LogService

## Purpose

Suppress repeated kernel warning/call-trace noise in LogService by rotating problematic `all.log` inputs and restarting `rsyslog` in COS.

## Procedure

1. Identify affected node(s) from warning message source in LogService.
1. On each affected node, move current log collector files out of `/var/log` active path:

```shell
mkdir -p /var/log/.all-log-backup
mv /var/log/all.log /var/log/.all-log-backup/
mv /var/log/all-*.log /var/log/.all-log-backup/ 2>/dev/null || true
```

1. Enter COS namespace and restart `rsyslog`:

```shell
ssh localhost -p20
systemctl restart rsyslog
```

1. Repeat on license node (`N10`) if it shows similar events.
1. Monitor EXAoperation LogService to confirm warning flood stops.

## Validation

- No continuous kernel call-trace warning spam in LogService.
- New log flow continues normally after `rsyslog` restart.

## Notes

- Keep backup logs until root-cause analysis is complete.
- This mitigates log flooding; underlying kernel/hardware cause may still require investigation.


