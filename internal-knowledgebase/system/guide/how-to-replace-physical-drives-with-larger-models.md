---
tool_name: cos
doc_type: guide
category: system
title: "How to replace physical drives with larger models"
summary: "Operational workflow to replace cluster disks with larger drives while preserving EXAStorage redundancy and restoring service safely."
---
# How to replace physical drives with larger models

## Purpose

Replace existing physical disks with larger models in a controlled way and restore EXAStorage redundancy.

## Preconditions

- All cluster components are online and stable.
- Affected volumes (non-temporary) have redundancy `>= 2`.
- No ongoing EXAStorage background recovery.
- The target disk is not the only OS disk.
- EXAStorage partition is the last partition on affected nodes.
- External backup has been created.

## Single-node replacement workflow

Use this sequence for one node at a time (multiple disks on same node can be handled together if redundancy allows).

1. Stop database(s).
2. Disable target disk in EXAStorage:

```shell
cshdd --disable -h /dev/sda6 -n 10
```

3. Stop EXAStorage and shut down affected node from EXAoperation.
4. Replace physical disk (and adjust RAID if required), then start node.
5. If required by your hardware workflow, stop EXAStorage/node again to finalize disk state before enlargement.
6. Start EXAStorage from EXAoperation.
7. Enlarge disk in EXAStorage:

```shell
cshdd --enlarge -h /dev/sda6 -n 10
```

8. Re-enable disk:

```shell
cshdd --enable -h /dev/sda6 -n 10
```

9. Wait until data restoration completes in EXAoperation/EXAStorage.
10. Repeat for other nodes.
11. Start database(s).

## Optional: topology visualization before multi-node replacement

```shell
csinfo --graph -i <ID> > ~/graph.dot
dot -T png -o ~/graph.png ~/graph.dot
```

Use graph output to verify redundancy sources remain available.

## De-duplication note

Detailed `cshdd` command reference is maintained in:

- `documents/cos/cos-storage-commands.md`

## Reference

- <https://docs.exasol.com/administration/on-premise.htm>


