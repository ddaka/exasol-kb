---
tool_name: internal-knowledgebase
doc_type: guide
category: database-sql
title: "Parallel CSV Import via FTP Using Named Pipes"
summary: "Set up a source-side FTP stream over named pipes and run parallel Exasol IMPORT FROM CSV operations for higher ingest throughput."
---

# Parallel CSV Import via FTP Using Named Pipes

## Overview

This guide describes a parallel file-ingest pattern where source exports stream into named pipes, exposed via FTP, and imported into Exasol in parallel.

## Architecture

- System A: Linux host with source database (for example PostgreSQL).
- System B: Exasol cluster.

## Prerequisites

- FTP service available on System A.
- Named pipe support on System A (`mkfifo`).
- Source-side export capability (for example `psql COPY ... TO STDOUT`).
- Target table DDL already created in Exasol.

## Procedure

### 1. Configure System A (source side)

Install/start an FTP server (example uses Twisted FTP):

```bash
sudo twistd ftp -p 22 -r /home/u/
```

Create one named pipe per parallel export stream:

```bash
mkfifo /home/u/output1.csv
mkfifo /home/u/output2.csv
```

Start source exports and redirect each stream to a dedicated named pipe:

```bash
PGPASSWORD='<password>' psql --username=postgres -w \
  --command="COPY (SELECT * FROM inventory WHERE prod_id % 2 = 0) TO STDOUT WITH CSV HEADER" \
  dellstore2 > /home/u/output1.csv

PGPASSWORD='<password>' psql --username=postgres -w \
  --command="COPY (SELECT * FROM inventory WHERE prod_id % 2 = 1) TO STDOUT WITH CSV HEADER" \
  dellstore2 > /home/u/output2.csv
```

## 2. Configure System B (Exasol side)

Create FTP connection object:

```sql
CREATE CONNECTION impcon TO 'ftp://192.168.56.12:22'
USER 'anonymous' IDENTIFIED BY 'me@example.com';
```

Run parallel import from both streams:

```sql
IMPORT INTO inventory FROM CSV
AT impcon FILE 'output1.csv'
AT impcon FILE 'output2.csv' SKIP = 1;
```

`SKIP = 1` skips one header line from each stream.

## Validation

- Confirm source exporters are actively writing into pipes.
- Verify Exasol import completes without stalled stream reads.
- Reconcile row counts against source partitions.

## Notes

- Ensure named pipe permissions allow FTP reader and database exporter access.
- Keep partition predicates mutually exclusive to avoid duplicate rows.
