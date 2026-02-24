---
tool_name: dwad_client
doc_type: guide
category: system
title: "Internal: Shrink a data volume (shell)"
summary: "Internal operational wrapper for dwad_client shrink-db with safety warnings and parameter mapping."
---
# Internal: Shrink a data volume (shell)

## Purpose

Shrink database volume size using shell command workflow.

## Warning

- Blocking and IO-intensive operation.
- Can cause significant temporary performance degradation.
- Final applied size may differ due to internal rounding.

## Command

```shell
dwad_client shrink-db <DB_NAME> <TARGET_MIB> <ON_PERSISTENT_VOLUME 1|0> <DRY_RUN 1|0>
```

Example (shrink `exa_test` from 10 GiB to 9 GiB on persistent volume, execute):

```shell
dwad_client shrink-db exa_test 9216 1 0
```

Parameter mapping:

- `9216` = target size in MiB
- `1` = persistent volume
- `0` = not dry-run

## De-duplication note

Canonical command reference:

- `documents/cos/dwad-client.md` (`shrink-db`, `abort-shrink`, `shrink-status`)


