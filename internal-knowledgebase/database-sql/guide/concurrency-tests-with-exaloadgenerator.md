---
tool_name: internal-knowledgebase
doc_type: guide
category: database-sql
title: "Run Concurrency Tests with EXAloadgenerator"
summary: "Use EXAloadgenerator to execute parallel SQL workloads, compare thread-level concurrency patterns, and analyze results using Exasol audit metadata."
---

# Run Concurrency Tests with EXAloadgenerator

## Overview

`EXAloadgenerator` is an internal command-line tool for running concurrency tests such as parallel query execution and import workloads over JDBC.

## Use Cases

- Evaluate behavior under increasing concurrent sessions.
- Compare query group performance at different thread counts.
- Generate repeatable benchmark runs with audit-visible client tags.

## Basic Usage

Display command options:

```bash
java -jar exaloadgenerator.jar --help
```

Minimal parallel run example:

```bash
java -jar exaloadgenerator.jar \
  -c jdbc:exa:10.60.120.16..19:8563 \
  -u sys \
  -P '<password>' \
  -d /tmp/querybenchmarkdir/ \
  -t 10
```

In this mode, each thread executes all query files in the directory.

## Advanced Usage

```bash
java -jar exaloadgenerator.jar \
  -c jdbc:exa:10.60.120.16..19:8563 \
  -u sys \
  -P '<password>' \
  -d /tmp/querybenchmarkdir/ \
  -t 10 \
  --executeEachQueryOnlyOnceGlobally \
  --minimumIdle 1 \
  --maximumIdle 2 \
  -tag myspecialbenchmark
```

Key behavior:

- `--executeEachQueryOnlyOnceGlobally`: each query file runs once globally across all threads.
- `--minimumIdle`/`--maximumIdle`: random think-time between statements.
- `-tag`: adds client tag for easier filtering in audit views.

## Batch Script Pattern

Example shell pattern for multiple thread counts:

```bash
#!/bin/bash

CONNECTION_STRING="10.60.13.11..15"
DB_USER="sys"
DB_PASSWORD="<password>"
RUN_TS="$(date +%s)"

for q_dir in SQL_1x/*; do
  [ -d "$q_dir" ] || continue

  for threads in 1 4 16; do
    run_tag="benchmark_${q_dir##*/}_${threads}_${RUN_TS}"

    java -jar ./exaloadgenerator.jar \
      -c "jdbc:exa:${CONNECTION_STRING}" \
      -u "${DB_USER}" \
      -P "${DB_PASSWORD}" \
      -d "${q_dir}" \
      -tag "${run_tag}" \
      -t "${threads}" \
      -f "${run_tag}.log"
  done
done
```

## Reporting Pattern

You can correlate runs by client tag in audit tables.

```sql
CREATE OR REPLACE VIEW benchmark_results_parallel AS
SELECT
    SECONDS_BETWEEN(s.stop_time, s.start_time) AS duration_in_seconds,
    REPLACE(REGEXP_SUBSTR(se.client, '(?i)Q[0-9]+'), 'Q', 'SQL') AS query_name,
    REPLACE(REGEXP_SUBSTR(se.client, '(?i)_[0-9]+_', 1, 2), '_', '') AS parallel,
    CAST(s.sql_text AS VARCHAR(5000)) AS sql_text
FROM EXA_DBA_AUDIT_SQL s
JOIN EXA_DBA_AUDIT_SESSIONS se USING (session_id)
WHERE s.command_name = 'SELECT'
  AND se.client LIKE '%<run_tag_fragment>%'
  AND s.sql_text NOT LIKE 'SELECT current_session'
ORDER BY s.start_time DESC;
```

## Notes

- Large thread counts can saturate client and network resources before the database becomes bottlenecked.
- Store credentials outside scripts where possible.
- Keep query files to one SQL statement per file for consistent execution behavior.

## Related

- [Concurrency Testing with EXAplus on Linux](concurrency-testing-using-exaplus-linux-shell.md)
