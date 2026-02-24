---
tool_name: internal-knowledgebase
doc_type: guide
category: database-sql
title: "How to save system tables after a PoC"
summary: "Export Exasol system tables and metadata after a PoC using the solutions-repository SQL helper, then import into a target database."
---
# How to save system tables after a PoC

## Purpose

Preserve system metadata/statistics from a PoC database before decommissioning the environment.

## Source script

Use SQL helper from Exasol solutions repository:

- <https://github.com/exasol/solutions/blob/master/Library/logging_and_debugging/generate_export_for_POC.sql>

## Constraints

- Exported objects are imported into one target schema.
- Target database version should match source version.

## Procedure

1. Run `generate_export_for_POC.sql` on source database.
2. Adjust parameters in the script header for your environment.
3. Execute generated `EXPORT` statements on source DB.
4. Collect generated files:
   - exported system table data files
   - `0_Overview.csv` (environment summary)
   - `0_Import.sql` (DDL + IMPORT statements)
5. Run `0_Import.sql` on destination DB to recreate/import exported data.

## Validation

- Verify expected system tables exist in target schema.
- Validate row counts for key exported tables.
- Keep `0_Overview.csv` with migration artifacts for traceability.


