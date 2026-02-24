---
tool_name: internal-knowledgebase
doc_type: guide
category: database-sql
title: "Derive Parquet File Structure and Use Test Data"
summary: "Guide for identifying unknown Parquet schema and sourcing sample Parquet files for import tests when customer data is unavailable."
---

# Derive Parquet File Structure and Use Test Data

## Overview

This guide covers two common PoC scenarios:

1. Parquet files exist, but schema is unknown.
2. No Parquet files are available for testing.

## Scenario 1: Unknown Parquet Schema

If file structure is unknown:

1. Download a sample Parquet file locally.
2. Inspect schema using a Parquet inspection tool (for example ParquetViewer).
3. Derive `CREATE TABLE` statement from the discovered schema.
4. Use the derived table definition before running import.

Tool reference:

- <https://github.com/mukunku/ParquetViewer>

Example workflow in ParquetViewer:

- `Edit -> Get SQL Create Table Script`

## Scenario 2: No Customer Parquet Data Available

If customer cannot provide files yet (for example export/write privileges missing), use public test files.

Test data source:

- <https://github.com/apache/parquet-testing>

Example file:

- <https://github.com/apache/parquet-testing/blob/master/data/int32_decimal.parquet>

This file provides a small integer-oriented dataset suitable for import pipeline validation.

## Validation Checklist

- Schema inferred from tool matches import target table.
- Data type mappings (precision/scale, nested fields if present) are reviewed before import.
- Small test import succeeds before processing production-sized files.
