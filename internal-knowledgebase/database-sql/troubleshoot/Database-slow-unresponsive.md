---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: database-sql
title: "Database slow or unresponsive"
summary: "Structured triage checklist for investigating severe performance degradation or unresponsive databases."
---
# Database slow or unresponsive

## Problem

A customer reports that the database is significantly slower than expected or unresponsive.

## Triage Checklist

Collect the following baseline information first:

- When the issue started and whether it is continuous or intermittent.
- Environment type and recent changes (configuration, workload, deployments).
- Whether backup/recovery operations are running or stuck.
- Whether storage volumes are degraded or recovering.
- Whether network interfaces show errors.
- Whether active session limits are reached or blocking sessions exist.
- Whether the operating system is responsive.
- Whether system/log partitions are near full.
- Whether swapping is active and huge pages are configured correctly.
- Whether any hardware incidents are present.

## Diagnostic Data Collection

Before restarting services, collect support data.

### Exasol 7

```shell
get_support_info -b 1 -e ANY_VALUE_DONT_CHANGE_ME -i 123
get_support_info -b 2 -e ANY_VALUE_DONT_CHANGE_ME -i 123
get_support_info -b 3 -e ANY_VALUE_DONT_CHANGE_ME -i 123
get_support_info -b 4 -e ANY_VALUE_DONT_CHANGE_ME -i 123
```

### Exasol 8

```shell
exasupport -b 1
exasupport -b 2
exasupport -b 3
exasupport -b 4
```

## Immediate Mitigation

1. Engage database support during business hours when possible.
1. If service impact is severe and no quick fix is identified, consider controlled restart of database or cluster after evidence collection.
1. Re-validate workload behavior after mitigation.

## Escalation Criteria

Escalate when:

- Root cause cannot be identified from initial triage.
- Performance remains degraded after controlled mitigation.
- Hardware, storage, or systemic failures are suspected.

## Result

The incident is resolved when performance returns to baseline and the root cause (or accepted mitigation) is documented.

---

_We welcome feedback on this troubleshooting article._
