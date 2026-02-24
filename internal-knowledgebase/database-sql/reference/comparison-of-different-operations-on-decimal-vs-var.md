---
tool_name: internal-knowledgebase
doc_type: reference
category: database-sql
title: "Benchmark Comparison: DECIMAL vs CHAR/VARCHAR Hash Keys"
summary: "Reference benchmark comparing filter, join, and aggregation behavior for DECIMAL IDs versus hash keys stored as CHAR/VARCHAR in older Exasol environments."
---

# Benchmark Comparison: DECIMAL vs CHAR/VARCHAR Hash Keys

## Scope

This document summarizes benchmark results comparing operations on:

- Numeric IDs (`DECIMAL(18,0)`)
- Hash-based keys stored as `CHAR(32)` / `VARCHAR(32)` (ASCII/UTF8 variants)

## Test Environment

- System: CTC1 (4+1)
- Exasol version: `6.0.beta3`
- Hardware: Dell 710 generation
- Per node: 24 cores, 240 GiB DB RAM
- Schema: RETAIL
- Main tables:
  - `SALES` (~300M rows)
  - `SALES_POSITIONS` (~3B rows)

Derived hash columns (from `HASH_MD5(sales_id)`):

- `SALES_ID_HASH_C_A` (`CHAR(32)` ASCII)
- `SALES_ID_HASH_VC_A` (`VARCHAR(32)` ASCII)
- `SALES_ID_HASH_C_U` (`CHAR(32)` UTF8)
- `SALES_ID_HASH_VC_U` (`VARCHAR(32)` UTF8)

## Operations Compared

1. Selective filter
2. Global aggregation
3. Global join
4. Local aggregation
5. Local join

## Duration Results (seconds)

All queries were executed twice (warm cache expectation).

| Operation | DECIMAL ID | CHAR(32) ASCII | VARCHAR(32) ASCII | CHAR(32) UTF8 | VARCHAR(32) UTF8 |
| --- | ---: | ---: | ---: | ---: | ---: |
| Filter | 1.600 | 17.689 | 16.750 | 16.779 | 16.679 |
| Global aggregation | 94.168 | 230.005 | 225.225 | 218.396 | 222.993 |
| Global join | 217.102 | 490.489 | 374.240 | 383.371 | 391.829 |
| Local aggregation | 46.723 | 129.543 | 142.321 | 130.083 | 129.381 |
| Local join | 14.859 | 62.361 | 36.449 | 44.172 | 39.621 |

## Resource Observations

- CPU/NET/TEMP behavior was generally similar across hash string variants.
- Main exception: aggregation TEMP usage.
  - DECIMAL-ID aggregation consumed significantly less TEMP than hash-key aggregation.

## Storage Observations

### Column sizes

| COLUMN_NAME | MEM_IN_GIB | RAW_IN_GIB |
| --- | ---: | ---: |
| SALES_ID | 11.73214900866151 | 25.19451661407948 |
| SALES_ID_HASH_C_A | 100.778842061758 | 100.7780664563179 |
| SALES_ID_HASH_C_U | 76.82251164875925 | 100.7780664563179 |
| SALES_ID_HASH_VC_A | 76.82115236762911 | 100.7780664563179 |
| SALES_ID_HASH_VC_U | 76.82144675590098 | 100.7780664563179 |

### Local index sizes

| INDEX_TABLE | REMARKS | MEM_IN_GIB |
| --- | --- | ---: |
| SALES_POSITIONS | LOCAL INDEX (SALES_ID) | 12.04565448500216 |
| SALES_POSITIONS | LOCAL INDEX (SALES_ID_HASH_C_A) | 18.39375721756369 |
| SALES_POSITIONS | LOCAL INDEX (SALES_ID_HASH_C_U) | 18.59675639122725 |
| SALES_POSITIONS | LOCAL INDEX (SALES_ID_HASH_VC_A) | 18.3937527006492 |
| SALES_POSITIONS | LOCAL INDEX (SALES_ID_HASH_VC_U) | 18.39356586057693 |

## Practical Takeaways

- In this benchmark, numeric ID-based operations were consistently faster than string-hash alternatives.
- `CHAR(32)` ASCII performed worst across tested scenarios.
- Later Exasol versions introduced hash datatype improvements that can change these tradeoffs.

## Caveat

These results were produced on older hardware and an older Exasol pre-release. Re-benchmark on your target version and workload before applying design decisions.

## Download

- [SQL_files.zip](https://github.com/exasol/Internal-Knowledgebase/files/9982766/SQL_files.zip)
