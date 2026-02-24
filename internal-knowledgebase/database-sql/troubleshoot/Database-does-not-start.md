---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: database-sql
title: "Database does not start"
summary: "Troubleshooting workflow for startup failures when nodes and EXAStorage appear healthy."
---
# Database does not start

## Problem

All required nodes are online and EXAStorage is running, but the database startup fails.

## Initial Checks

1. Confirm the data volume is unlocked and accessible.
1. Validate hardware health on all nodes.

```shell
ipmitool sel elist
dmesg
```

If available, also validate vendor tools such as OMSA or MegaCLI.

## Diagnostic Procedure

1. Collect logs for core components:

```shell
logd_collect EXAoperation
logd_collect Storage
logd_collect Load
logd_collect EXASolution_<DB_NAME>
```

1. Identify the startup stage where the failure occurs.

```shell
watch cospstree
```

1. Check whether configuration or parameter changes were made recently.
1. Inspect database process logs for backtraces and startup errors on logical node 0.

```shell
dwad_client sys-nodes <DB_NAME>
```

- Exasol 8 log path: `/exa/logs/db/`
- Exasol 7 log path: `/d02_data/<DB_NAME>/log/process/`
- Key processes: `Controller`, `Pddserver`, `Objectserver`, `Sqllogserver`, `Connectionserver`

## Remediation

1. If a specific node is identified as faulty, replace it with a reserve node and retry startup.
1. Re-test database startup and validate client connectivity.
1. If the issue cannot be isolated, escalate with collected logs and findings.

## Result

The incident is resolved when the database starts successfully and remains stable after restart.

---

_We welcome feedback on this troubleshooting article._
