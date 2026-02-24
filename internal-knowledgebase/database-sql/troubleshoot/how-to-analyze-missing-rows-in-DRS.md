---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: database-sql
title: "How to analyze missing rows in DRS"
summary: "Step-by-step investigation for source/target row count mismatches in Data Replication Service (DRS)."
---
# How to analyze missing rows in DRS

## Problem

A replicated target table contains fewer rows than the source table long after expected replication delay.

## Scope

This procedure focuses on SQL-level analysis in source, target, and DRS repository objects.

## Prerequisites

- SQL access to source and target databases.
- SQL access to DRS repository schema.
- `SELECT` on required DRS repository tables.
- `SELECT` on the affected table in source/target.
- `SELECT ANY DICTIONARY` (or exported audit data) for audit-based analysis.

## Step 1: Validate replication rule and plan

Check matching rule, plan, business key, and timestamp column.

```sql
select *
from SYNC_REPOSITORY.SYNC_RULE
where 'PROBLEM_TABLE' regexp_like TABLE_MATCH
order by RULE_PRIORITY desc
limit 10;
```

```sql
select * from SYNC_REPOSITORY.SYNC_PLAN where PLAN_ID = <SYNC_PLAN>;
select * from SYNC_REPOSITORY.SYNC_KEY  where KEY_ID  = <BUSINESS_KEY>;
```

Confirm:

- Effective rule priority is valid.
- Plan mode matches expectation (`MERGE` if delete detection is required).
- Plan frequency/weight are reasonable.
- Business key and timestamp column definitions are correct.

## Step 2: Inspect table replication history

```sql
select SL.SCHEMA_NAME, TL.*
from SYNC_REPOSITORY.JOB_SCHEMA_LOG as SL
join SYNC_REPOSITORY.JOB_TABLE_LOG as TL using (JOB_SCHEMA_ID)
where TABLE_NAME = 'PROBLEM_TABLE'
order by EXPORT_START desc
limit 20;
```

Analyze for anomalies:

- Overlapping/near-simultaneous exports on the same table.
- `ROWS_EXPORTED` inconsistent with observed source growth.
- `MODIFIED_FROM` / `MODIFIED_UPTO` windows that skip expected data ranges.
- `TABLE_SIZE` diverging from target row count over time.

## Step 3: Compare source and target row counts

```sql
select count(*) from PROBLEM_SCHEMA.PROBLEM_TABLE;
```

Run on both source and target.

If target is smaller, compare counts for historical timestamp windows from `JOB_TABLE_LOG`:

```sql
select count(*)
from PROBLEM_SCHEMA.PROBLEM_TABLE
where LAST_UPDATE_TS <= '<MODIFIED_UPTO_TIMESTAMP>';
```

This helps determine whether late/out-of-order timestamp writes caused rows to miss delta windows.

## Step 4: Verify ETL behavior via auditing

Use `EXA_DBA_AUDIT_SQL` to correlate ETL DML and DRS export timing.

```sql
select SESSION_ID, STMT_ID, COMMAND_NAME, START_TIME, STOP_TIME, ROW_COUNT, SQL_TEXT
from EXA_DBA_AUDIT_SQL
where START_TIME >= '<start>' and START_TIME <= '<end>'
  and COMMAND_CLASS = 'DML'
  and instr(SQL_TEXT, 'PROBLEM_TABLE') > 0
order by START_TIME;
```

Look for:

- Large insert chains with intermediate commits.
- DRS exports running while ETL is still actively inserting.
- Timestamps not maintained in strictly increasing order for replicated rows.

## Likely root cause pattern

A common cause is ETL loading data in chunks with intermediate commits while reusing source timestamps that are not monotonic for DRS delta logic.

Result: DRS windowing by timestamp can miss rows even though later table-size snapshots appear larger.

## Remediation options

- Update ETL logic to maintain a reliable technical update timestamp for replication.
- Remove or reduce intermediate commits where feasible.
- Review sync key configuration to avoid unsupported duplicates.
- Reinitialize or repair affected table replication state if required.

## Notes

- This is often a data/timestamp contract issue, not a DRS engine defect.
- Keep customer communication factual and evidence-based.

## References

- <https://github.com/exasol/data-replication-service/issues/282>
- <https://github.com/exasol/data-replication-service/issues/40>

---

_We welcome feedback on this troubleshooting article._
