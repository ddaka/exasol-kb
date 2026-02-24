---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: database-sql
title: "Database Failsafety Alert"
summary: "Investigate and resolve a critical Database Failsafety monitoring alert."
---
# Database Failsafety Alert

## Problem

A monitoring alert is raised with details similar to:

**Case Subject:** Database Failsafety
**Case Description:** `Database Failsafety is: crit state: FAILSAFETY detected`

The alert tags usually include the affected `clusterID`, account group, and database name.

## Procedure

1. Identify the affected database from the alert tags.
1. Check the current database status.
1. Verify whether the database was recently restarted.
1. If the database restarted, inform the customer and start root cause analysis (RCA).
1. If the database is stopped, start the database first, then continue with RCA.
1. If the database is running but the alert remains active, collect relevant logs and continue incident investigation.

## Customer Communication

When customer communication is required, include:

- The affected database name.
- Current service state (running/stopped/restarted).
- Immediate mitigation already performed.
- Next RCA update timeline.

## Result

The alert is resolved when database failsafety is no longer triggered and the service state is stable.

---

_We welcome feedback on this troubleshooting article._
