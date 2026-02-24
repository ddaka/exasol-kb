---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: database-sql
title: "Database Uptime Monitor Alert"
summary: "Handle alerts indicating a recent database or host restart detected by uptime monitoring."
---
# Database Uptime Monitor Alert

## Problem

A monitoring alert is raised with details similar to:

**Case Subject:** Database Uptime Monitor
**Case Description:** `System probably restarted a few minutes ago`

This indicates a recent restart event that may have affected database availability.

## Procedure

1. Identify the affected database and cluster from alert tags.
1. Check whether the database is currently online and healthy.
1. Confirm whether the restart was planned (maintenance/release) or unexpected.
1. If the restart was unplanned, notify the customer and open RCA.
1. If the database is stopped, start it and verify normal operation.
1. Collect restart timeline and relevant logs for incident documentation.

## Customer Communication

For unplanned events, include:

- Restart timestamp.
- Current database state.
- Service impact window.
- Next RCA update timeline.

## Result

The alert is resolved when the database is stable after restart and the root cause workstream is initiated when required.

---

_We welcome feedback on this troubleshooting article._
