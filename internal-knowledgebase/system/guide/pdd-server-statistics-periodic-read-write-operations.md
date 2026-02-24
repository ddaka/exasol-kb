---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "PDD Server statistics (periodic read write operations)"
summary: "Estimate hourly disk IO pressure by converting PDD periodic read/write metrics into time and percentage of one hour."
---
# PDD Server statistics (periodic read write operations)

## Purpose

Use PDD periodic metrics to estimate how much time each node spends on disk read/write work per hour.

## Interpretation limits

This signal is directional only. High percentages can indicate IO pressure, but always correlate with query patterns, wait states, and system metrics.

## Locate PDD log records

PDD periodic records are typically emitted about once per hour per node.

Current COS-oriented path:

```bash
ls -1 /exa/logs/db/<DB_NAME>/*PddServer* /exa/logs/db/<DB_NAME>/pddserver-*.log 2>/dev/null
```

Legacy EXAoperation-style deployments may store process logs under:

```text
/d02_data/<DB_NAME>/log/process
```

Extract periodic read/write lines:

```bash
grep -Ehi 'hdd (read|write) periodic' /exa/logs/db/<DB_NAME>/* 2>/dev/null | grep -vi sync
```

Example output:

```text
05.03 04:16:29.979   hdd read periodic:  179524.61 MB |   286.58 MB/s
05.03 04:16:29.979   hdd write periodic: 152991.72 MB |   337.19 MB/s
```

## Calculation method

For each metric line:

- `seconds_in_io = MB / (MB_per_second)`
- `hour_percentage = (seconds_in_io / 3600) * 100`

### Read percentage example

Input:
- `read_mb = 179524.61`
- `read_mb_s = 286.58`

Result:

```text
179524.61 / 286.58 = 626.44 s
(626.44 / 3600) * 100 = 17.40%
```

### Write percentage example

Input:
- `write_mb = 152991.72`
- `write_mb_s = 337.19`

Result:

```text
152991.72 / 337.19 = 453.73 s
(453.73 / 3600) * 100 = 12.60%
```

## Quick calculator snippet

```bash
echo "179524.61 286.58" | awk '{sec=$1/$2; printf "seconds=%.2f hour_pct=%.2f%%\n", sec, (sec*100/3600)}'
```

## De-duplication note

Canonical process and log context:

- `documents/cos/cos-database-partitions.md`
- `documents/cos/cos_directory0structure.md`
