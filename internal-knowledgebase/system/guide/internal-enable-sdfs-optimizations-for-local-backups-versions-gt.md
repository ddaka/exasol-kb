---
tool_name: sdfs
doc_type: guide
category: system
title: "Internal: Enable SDFS optimizations for local backups (versions >= 6.0.2)"
summary: "Internal procedure to format archive volumes with larger SDFS partition size for local backup optimization."
---
# Internal: Enable SDFS optimizations for local backups (versions >= 6.0.2)

## Purpose

Apply recommended SDFS partition sizing for large local archive volumes to improve backup behavior.

## Preconditions

- Use a new archive volume (volume formatting required).
- Ensure no backup is running before freeze/format.
- Understand sizing effect: volume size rounds to a multiple of `NR_OF_HDDS_PER_NODE * NR_OF_NODES * 256 GiB`.

## Procedure

1. Freeze volume before formatting:

```shell
csvol -E -v <VOLID> -t all
```

2. Format volume with 256 GiB partition size:

```shell
SDFS_PARTITION_SIZE_BYTES=$((256*1024*1024*1024)) sdfs format <VOLUME_ID>
```

## Notes

- If 256 GiB partition sizing is not acceptable, use a smaller partition size.
- This is an internal tuning recommendation for specific legacy versions.

## De-duplication note

General SDFS volume operations are documented in:

- `documents/cos/sdfs-overview-and-volumes.md`


