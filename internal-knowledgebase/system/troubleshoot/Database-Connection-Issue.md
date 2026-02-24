---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Database Connection Issue Alert"
summary: "Operational response checklist for Database Invalid Connection alerts, including database reachability checks, session-capacity validation, and Proust partition recovery steps."
---

# Database Connection Issue Alert

## Alert Pattern

Typical case subject:

- `Database Connection Issue`

Typical case description:

- `Database Invalid Conn`

## Interpretation

Monitoring indicates it cannot reliably connect to the target database.

Use alert tags to identify:

- Cluster ID
- Account group
- Affected database

## Response Procedure

1. Confirm Proust has current data.
2. Check database state/availability.
3. Log in to the database and run a simple validation query.
4. Verify session capacity (maximum active sessions not exhausted).
5. If issue persists, restart the relevant Proust partition.
6. If still unresolved:
   - notify team in `proust-feedback`
   - silence alert temporarily according to incident policy

## Validation

- Monitoring connectivity restored.
- Validation query succeeds.
- Alert clears or stays healthy after silence window expires.
