---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: database-sql
title: "Database Startup Alert"
summary: "Investigate database startup alerts and determine whether the restart was planned or unexpected."
---
# Database Startup Alert

## Problem

A monitoring alert is raised with details similar to:

**Case Subject:** Database Startup
**Case Description:** `Database Startup detected`

This alert indicates that a database startup event was detected for the monitored environment.

## Procedure

1. Identify the affected database from the alert tags (`clusterID`, account group, database name).
1. Check the current database status.
1. Verify whether the startup is expected (planned maintenance, deployment, or approved operation).
1. If the startup was expected, document the event and close the incident if no further issue exists.
1. If the startup was unexpected, inform the customer and open RCA.
1. Collect timeline and logs required for incident analysis.

## Customer Communication

For unplanned startup events, include:

- Affected database and time of startup.
- Current availability state.
- Whether service impact occurred.
- Next RCA update timing.

## Result

The alert is resolved when startup cause is confirmed, customer communication is completed (if required), and no ongoing instability is detected.

---

_We welcome feedback on this troubleshooting article._
