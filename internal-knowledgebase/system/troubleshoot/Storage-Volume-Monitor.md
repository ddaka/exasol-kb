---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Storage Volume Monitor"
summary: "An alert was created as follows:"
---
# Storage Volume Monitor

## Problem

An alert was created as follows:

Case Subject: Storage Volume Monitor

Case Description:

- archive volume with ID X state: Degraded
- archive volume with ID X state: Locked
- persistent volume with ID X state: Locked
- persistent volume with ID X state: Degraded
## Procedure

Volumes with state Degraded:

When one of the data nodes fails, the failover is triggered and reserve node will take over the role from the failed node. Yet, some segment are still resided on other data nodes, which causes DEGRADED state of the volume and less performant situation.

In order to fix it, data segments should be moved from the faulty node to the reserve node.

More info: [Swap Active/Reserve Node](https://docs.exasol.com/db/7.1/administration/on-premise/nodes/replace_active_node.htm)

Volumes with state LOCKED:

Usually, when the redundancy of the volumes is 1 or one of the nodes fails and there is a missing reserve node, the state of the volume will be LOCKED. In this case, if this is a persistent volume, the database will not be available. For archive volumes, the backups are not available and a new backup can not be triggered.

In order to fix it, the faulty node should be recovered when Redundancy is 1 or integrate a reserve node if available when Redundancy is 2.

## Additional References


