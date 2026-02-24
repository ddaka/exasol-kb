---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "Internal: Calculate additional space for cluster enlargement"
summary: "Estimate per-node additional storage needed before cluster enlargement using table and index memory footprints."
---
# Internal: Calculate additional space for cluster enlargement

## Purpose

Estimate required free storage per node before enlarging a cluster (example: `4+1` to `8+1`).

## Input parameters

- Current active node count (`OLD_NODECOUNT`)
- Target active node count (`NEW_NODECOUNT`)
- Current table memory footprint
- Current index memory footprint

## Core formula

```text
REQUIRED_HDDFREE_IN_MB = (TABLE_MEMSIZE_IN_MB + INDEX_MEMSIZE_IN_MB) / OLD_NODECOUNT
```

## Example SQL

```sql
define OLD_NODECOUNT=4;
define NEW_NODECOUNT=8;

select
    rpad('"' || S2_SN || '"."' || S2_TN || '"', 90) TABLE_NAME,
    (TABLE_MEMSIZE_IN_MB + zeroifnull(INDEX_MEMSIZE_IN_MB)) / &OLD_NODECOUNT AS REQUIRED_HDDFREE_IN_MB
from
    (
        select
            ROOT_NAME as S2_SN,
            OBJECT_NAME as S2_TN,
            cast(zeroifnull(sum(MEM_OBJECT_SIZE) / 1024 / 1024) as dec(18, 1)) TABLE_MEMSIZE_IN_MB
        from EXA_DBA_OBJECT_SIZES
        where OBJECT_TYPE = 'TABLE'
        group by 1, 2
    ) S2
full outer join
    (
        select
            INDEX_SCHEMA as S3_SN,
            INDEX_TABLE as S3_TN,
            cast(zeroifnull(sum(MEM_OBJECT_SIZE) / 1024 / 1024) as dec(18, 1)) INDEX_MEMSIZE_IN_MB
        from "$EXA_INDICES"
        group by 1, 2
    ) S3
on S2_SN = S3_SN and S2_TN = S3_TN
order by REQUIRED_HDDFREE_IN_MB;
```

## Recommendation

If temporary space is limited during enlargement, reorganize smaller tables first to release capacity progressively.


