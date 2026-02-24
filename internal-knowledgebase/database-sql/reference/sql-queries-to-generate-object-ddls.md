---
tool_name: internal-knowledgebase
doc_type: reference
category: database-sql
title: "SQL Queries to Generate Object DDLs"
summary: "Internal reference for generating table/view DDLs directly via SQL when support has dictionary access on customer systems."
---

# SQL Queries to Generate Object DDLs

## Purpose

Support and development workflows often require object DDL (tables/views and dependencies) for troubleshooting.

## Existing Customer-Facing Approaches

- <https://exasol.my.site.com/s/article/Create-DDL-for-a-table>
- <https://exasol.my.site.com/s/article/How-to-create-DDL-for-Exasol-support>

## Internal SQL-Only Approach

When support has direct cluster access and required dictionary privileges (`SELECT ANY DICTIONARY`), internal SQL-only scripts can generate DDL without asking customers to run external scripts.

Internal script references:

- [create_table_ddl_pure_sql.sql](https://github.exasol.com/integrated-professional-services/customer-specifics/blob/master/Support/create_table_ddl_pure_sql.sql)
- [create_view_ddl_pure_sql.sql](https://github.exasol.com/integrated-professional-services/customer-specifics/blob/master/Support/create_view_ddl_pure_sql.sql)

## Notes

- These internal SQL queries are not intended as public/customer documentation.
- Customer-facing scripts remain easier to maintain and explain externally.
- Future built-in functionality may supersede this approach:
  - <https://exasol.atlassian.net/browse/SPOT-14983>
