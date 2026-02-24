---
tool_name: confd_client
doc_type: guide
category: system
title: "Resize local SDFS archive volume"
summary: "Calculate block requirements and safely enlarge local SDFS archive volume capacity using st_volume_enlarge/csresize workflows."
---
# Resize local SDFS archive volume

## Purpose

Enlarge a local SDFS archive volume safely.

## Critical warning

This is a high-risk storage operation. Incorrect block-size planning can cause backup data loss.

## Block conversion formulas

Blocks -> GiB:

```text
blocks / master_nodes / block_size / stripe_size = GiB
```

GiB -> total blocks:

```text
GiB * master_nodes * block_size * stripe_size = total_blocks
```

For enlarge operations, use **blocks per node**.

Example (add 32 GiB, 4 master nodes, block=64, stripe=64):

```text
(32 * 4 * 64 * 64) / 4 = 131072 blocks_per_node
```

## Max archive volume size by block size

| Block size | Max volume size |
| --- | --- |
| 64 KiB | 16 TB |
| 128 KiB | 32 TB |
| 256 KiB | 64 TB |
| 512 KiB | 128 TB |
| 1024 KiB | 256 TB |

Block size cannot be changed later.

## Procedure

1. Inspect current volume parameters:

```shell
csinfo -v -i <VID>
```

2. Check SDFS limits/debug info:

```shell
sdfs debug <VID>
```

3. Ensure no backup/restore is running, then lock/freeze volume:

```shell
csvol -E -t all -v <VID>
```

4. Enlarge volume (preferred on v8):

```shell
confd_client st_volume_enlarge {blocks_per_node: <BLOCKS_PER_NODE>, vid: <VID>}
```

Legacy alternative:

```shell
csresize -e -v <VID> -b <BLOCKS_PER_NODE>
sdfs resize <VID>
```

5. Verify new size:

```shell
csinfo -v -i <VID>
```

## De-duplication note

Canonical command references:

- `documents/cos/confd-volume-management.md` (`st_volume_enlarge`)
- `documents/cos/cos-storage-commands.md` (`csresize`)


