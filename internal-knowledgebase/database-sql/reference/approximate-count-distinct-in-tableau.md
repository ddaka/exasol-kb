---
tool_name: internal-knowledgebase
doc_type: reference
category: database-sql
title: "Use APPROXIMATE_COUNT_DISTINCT in Tableau"
summary: "Tableau does not expose Exasol APPROXIMATE_COUNT_DISTINCT directly; use a RAWSQL aggregate expression to call it."
---

# Use APPROXIMATE_COUNT_DISTINCT in Tableau

## Question

Can `APPROXIMATE_COUNT_DISTINCT` be used from Tableau?

## Answer

Not as a native Tableau aggregation. Use a custom calculation with `RAWSQLAGG_INT`:

```sql
RAWSQLAGG_INT("APPROXIMATE_COUNT_DISTINCT(%1)", [<column_name>])
```

## Notes

- Replace `<column_name>` with the Tableau field to estimate distinct values.
- Validate expected precision behavior for your use case.

## Reference

- <https://docs.exasol.com/sql_references/functions/alphabeticallistfunctions/approximate_count_distinct.htm>
