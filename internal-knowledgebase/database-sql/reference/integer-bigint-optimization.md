---
tool_name: internal-knowledgebase
doc_type: reference
category: database-sql
title: "INTEGER/BIGINT optimization"
summary: "Reference for choosing efficient DECIMAL precision when migrated integer columns do not require full source range."
---
# INTEGER/BIGINT optimization

## Purpose

Migrated schemas often use larger numeric precision than required, which can increase query runtime and temporary memory usage (especially for joins and `COUNT(DISTINCT ...)`).

## Background

During heterogeneous migrations, integer-like types are commonly mapped conservatively:

| Source type | Typical safe Exasol mapping | Full-range minimum |
| --- | --- | --- |
| INTEGER (32-bit) | `DECIMAL(10)` | `DECIMAL(10)` |
| BIGINT (64-bit) | `DECIMAL(19)` | `DECIMAL(19)` |

If real data never uses full range, reducing precision can improve execution efficiency.

## Practical thresholds

- `DECIMAL(p<=9, s)` can fit 32-bit internal representation.
- `DECIMAL(p<=18, s)` can fit 64-bit internal representation.

Common optimization pattern:

- `DECIMAL(10)` -> `DECIMAL(9)` when data range proves safe.
- `DECIMAL(19)` -> `DECIMAL(18)` when data range proves safe.

## Expected impact

- Can reduce `RAW_OBJECT_SIZE`.
- Can improve join/group/distinct performance.
- `MEM_OBJECT_SIZE` improvement may be limited depending on workload.

## Validation before change

1. Measure actual min/max values on source data.
1. Confirm no future load can exceed proposed precision.
1. Benchmark representative queries before/after type change.

## Tooling reference

- <https://github.com/EXASOL/database-migration/tree/master/post_load_optimization>


