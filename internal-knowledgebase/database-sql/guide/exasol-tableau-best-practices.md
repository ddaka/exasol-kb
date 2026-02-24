---
tool_name: internal-knowledgebase
doc_type: guide
category: database-sql
title: "Exasol and Tableau Best Practices"
summary: "Practical guidance for building performant Tableau dashboards on Exasol, including modeling, query-shaping, and large-dataset design patterns."
---

# Exasol and Tableau Best Practices

## Overview

This guide summarizes practical patterns for running Tableau workloads efficiently on Exasol, especially for large datasets and complex dashboards.

## Tableau Design Practices

### Start at aggregate level

- Begin with summary-level views.
- Use interactions (actions and drill paths) to move to detail only when needed.
- Avoid loading very large raw result sets directly into one visualization.

### Avoid overusing quick filters

Large dashboards with many quick filters often generate heavy SQL patterns and unnecessary query volume.

Preferred approach:

- Use Tableau actions and guided drill flows.
- Reduce filter complexity where possible.

### Review LOD-heavy models

`LOD` (Level of Detail) calculations can generate expensive SQL.

- Review encapsulated/nested LOD expressions.
- Move stable heavy logic to database-side preprocessing where appropriate.
- For older deployments, validate whether null-join behavior and constraints can be optimized.

## Exasol-Side Practices

### Use relational constraints where valid

Define foreign keys and `NOT NULL` constraints where data guarantees exist.

Benefits:

- Better optimizer decisions.
- Potential join elimination/culling in BI-generated SQL.

### Align join datatypes

Ensure join columns use compatible datatypes to avoid implicit casts and degraded plans.

### Apply standard data model tuning

- Optimize datatypes where possible.
- Use suitable distribution keys and replication boundaries.
- Precompute selective lookup tables when they reduce repeated runtime expressions.

## Large Dataset Guidance

- Do not render extremely high-cardinality charts/maps without pre-aggregation.
- Avoid deep crosstab drill structures over unbounded row sets.
- Prefer progressive exploration patterns (summary -> filtered detail).

## Downloads

- [preprocess_NULLJOIN.sql](https://github.com/exasol/internal-knowledgebase/blob/main/Connect-with-Exasol/attachments/preprocess_NULLJOIN.sql)
- [Module_TableauBestPractices_rebranded.zip](https://github.com/exasol/Internal-Knowledgebase/files/9981689/Module_TableauBestPractices_rebranded.zip)

## References

- [convert_datatypes.sql](https://github.com/exasol/database-migration/blob/master/post_load_optimization/convert_datatypes.sql)
