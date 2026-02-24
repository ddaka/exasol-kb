---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "Internal - How to calculate hugepages"
summary: "Estimate hugepage memory target per node based on RAM, OS reservation, and configured system heap limits."
---
# Internal - How to calculate hugepages

## Purpose

Calculate recommended hugepage allocation per data node.

## Important

Applying RAM/hugepage configuration changes requires restart of all data nodes.

## Inputs required

- `RAM_PER_NODE` (MB): from `free -m` total memory.
- `OS_RESERVED` (MB): recommended `10%` of `RAM_PER_NODE`.
- `Heap_MAX_1..N` (MB): max system/process heap per active DB (use largest configured value or version default).

## Formula

```text
CALC_HP = RAM_PER_NODE - OS_RESERVED - SUM(Heap_MAX_1..N)
REAL_HP = (CALC_HP / RAM_PER_NODE * 100) > 50 ? CALC_HP : 0
```

Rules:

- If `CALC_HP` is less than 50% of `RAM_PER_NODE`, set hugepages to `0`.
- If `RAM_PER_NODE < 96 GiB`, do not use hugepages.

## Reference calculator

- <https://jscalc.io/calc/TzmsQabm1r9Ut7zV>

## Example reference values

| RAM/node (GB) | REAL_HP (GB) | REAL_HP ratio |
| --- | --- | --- |
| 768 | 660 | 86% |
| 512 | 429 | 84% |
| 384 | 314 | 82% |
| 256 | 198 | 77% |
| 192 | 140 | 73% |
| 128 | 84 | 66% |
| 96 | 56 | 56% |
| 72 | 0 | 44% |
| 64 | 0 | 40% |


