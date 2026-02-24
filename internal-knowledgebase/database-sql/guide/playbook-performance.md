---
tool_name: internal-knowledgebase
doc_type: guide
category: database-sql
title: "Playbook: performance analysis entry point"
summary: "Classify performance requests and route to the correct Exasol analysis playbook with required inputs."
---
# Playbook: performance analysis entry point

## Purpose

Use this page to classify performance requests and choose the right analysis flow.

## Classification model

Classify along two dimensions:

1. Scope:
- `system`: database-wide behavior (jobs, reports, overall throughput).
- `query`: specific SQL statement(s).

1. Nature:
- `absolute`: objectively slow / unmet expectation.
- `relative`: degraded vs previous baseline.

## Required inputs by class

### System + absolute

Required:

- Time window.
- KPI definition for slow vs acceptable.
- Data source: logs, exported stats, or dictionary access.

Route: `playbook-performance-system-absolute.md`

### System + relative

Required:

- Two comparable time windows (fast vs slow).
- Same KPI and evidence sources as above.

Route: `playbook-performance-system-relative.md`

### Query + absolute

Required:

- `(session_id, stmt_id)` for slow query.
- Query profile extract.

Helpful:

- Session logs.
- SQL text and related DDL for deeper optimization.

Route: `playbook-performance-query-absolute.md`

### Query + relative

Required:

- Slow and fast `(session_id, stmt_id)` samples.
- At least one profile from each class.

Helpful:

- `EXA_SQL_LAST_DAY` / audit extracts.
- Logs and SQL/DDL artifacts.

Route: `playbook-performance-query-relative.md`

## Practical triage checklist

1. Confirm scope (`system` vs `query`).
1. Confirm nature (`absolute` vs `relative`).
1. Verify required evidence exists.
1. Route to target playbook.
1. If evidence is missing, request it before deep analysis.

## Related references

- Query profile extraction procedure.
- Support package/log collection procedure.
- Statistics export collection procedure.
- Query/view DDL extraction procedure.


