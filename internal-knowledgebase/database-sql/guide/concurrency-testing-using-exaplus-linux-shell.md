---
tool_name: internal-knowledgebase
doc_type: guide
category: database-sql
title: "Concurrency Testing with EXAplus on Linux"
summary: "Run synchronized parallel SQL workloads with multiple EXAplus sessions using a FIFO barrier and screen-based process control."
---

# Concurrency Testing with EXAplus on Linux

## Overview

This guide describes a lightweight way to run concurrent SQL tests with multiple `exaplus` sessions, synchronized through a FIFO barrier.

## When to Use This Method

Use this method when you need quick, scriptable concurrency checks without additional load-testing tools.

Alternative tools for larger test programs:

- JMeter
- EXAloadgenerator

## Limitations

- Each `exaplus` process consumes local client memory.
- This pattern is suitable for synchronized-start tests, not long-running closed-loop throughput models.

## Test Concept

- Start multiple `exaplus` processes in parallel.
- Make each process block on `IMPORT FROM LOCAL CSV FILE` reading the same FIFO.
- Release all sessions at once by writing a single line into the FIFO.
- Measure execution time with `TIMER` and spool files.

## Procedure

### 1. Create a synchronization FIFO

```bash
mkfifo the_fifo.csv
```

### 2. Prepare `sync.sql`

```sql
DEFINE FILE_NAME = &1;
SPOOL &FILE_NAME..txt;

-- Synchronization barrier
IMPORT INTO (tmp VARCHAR(20))
FROM LOCAL CSV FILE 'the_fifo.csv';

TIMER START '&FILE_NAME';
@&FILE_NAME;
TIMER STOP '&FILE_NAME';
```

### 3. Start concurrent sessions in `screen`

```bash
screen
for FILE in /path/to/sql/*.sql; do
  screen exaplus --profile db_login -f sync.sql -- "$FILE"
done
```

Wait until sessions are blocked on the FIFO import.

### 4. Release all sessions simultaneously

```bash
echo "start" > the_fifo.csv
```

### 5. Monitor and clean up

- Watch `screen` windows until all sessions finish.
- Review generated spool/timer outputs.
- Remove FIFO:

```bash
rm -f the_fifo.csv
```

## Validation

- All sessions start actual workload at approximately the same time.
- Timer and spool outputs are generated for each workload file.
- No local resource saturation on the client host during test execution.
