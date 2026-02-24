---
tool_name: c4
doc_type: reference
category: system
title: "Which script language containers are still in use"
summary: "Determine whether older Script Language Container (SLC) packages can be safely removed from BucketFS."
---
# Which script language containers are still in use

## Purpose

Identify whether older Script Language Container (SLC) files are still referenced before deleting them to reclaim disk space.

## Background

- Exasol UDF runtimes use SLC packages.
- In Exasol 8, standard SLCs are managed as c4 packages with controlled version retention.
- In older setups, SLC archives in BucketFS can accumulate across upgrades.

## Decision flow

### 1. Identify active standard SLC

```sql
SELECT *
FROM exa_commandline
WHERE param_name = 'builtinScriptLanguageName';
```

The returned SLC is the currently active standard container.

### 2. Check system-level language mapping

```sql
SELECT *
FROM exa_parameters
WHERE parameter_name = 'SCRIPT_LANGUAGES';
```

If mappings only reference builtin aliases and not old BucketFS SLC files, those old files are likely unused at system level.

### 3. Check session-level overrides

```sql
SELECT *
FROM exa_dba_audit_sql
WHERE command_name = 'ALTER SESSION'
  AND sql_text LIKE '%SCRIPT$_LANGUAGES%' ESCAPE '$'
ORDER BY start_time DESC;
```

If audit history shows explicit references to old SLC files, do not remove them until usage is clarified.

## Removal criteria

Candidate SLC files are generally safe to remove only when all of the following are true:

- Not the current `builtinScriptLanguageName`.
- Not referenced in `SCRIPT_LANGUAGES` system parameter.
- Not referenced by session-level overrides (or auditing confirms no such usage).

## Limitations

If SQL auditing is disabled, absence of evidence in `EXA_DBA_AUDIT_SQL` is not proof of non-usage.

## References

- <https://docs.exasol.com/db/latest/database_concepts/udf_scripts/adding_new_packages_script_languages.htm>
- <https://docs.exasol.com/db/latest/administration/aws/admin_interface/c4.htm>
- <https://docs.exasol.com/db/latest/sql_references/system_tables/statistical/exa_dba_audit_sql.htm>


