---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: database-sql
title: "Database Monitor Alert"
summary: "Investigate and resolve alerts indicating that a monitored database is offline."
---
# Database Monitor Alert

## Problem

A monitoring alert is raised with details similar to:

**Case Subject:** Database Monitor
**Case Description:** `Database Offline`

The alert tags usually identify the affected `clusterID`, account group, and database name.

## Procedure

1. Confirm the impacted database from the alert metadata.
1. Check current database state.
1. Verify whether the database was recently restarted.
1. If it was restarted, inform the customer and start RCA.
1. If it is stopped, start the database and continue RCA.
1. If it is running but the alert is still active, collect logs and continue investigation.

## Customer Communication

When contacting the customer, include:

- Affected database and cluster.
- Current service state.
- Actions already taken.
- Expected next RCA update.

## Result

The incident is resolved when the database is online and the monitoring alert is cleared.

---

_We welcome feedback on this troubleshooting article._
