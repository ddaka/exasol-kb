---
tool_name: sdfs
doc_type: guide
category: system
title: "Internal: Enable SDFS optimizations for remote backups (versions >= 6.0.2)"
summary: "Internal procedure to format archive volumes with larger partition size and enable search/init labels for remote backup performance."
---
# Internal: Enable SDFS optimizations for remote backups (versions >= 6.0.2)

## Purpose

Apply internal SDFS tuning for remote-backup scenarios with large archive volumes.

## Preconditions

- Use a new archive volume.
- Ensure no backup jobs are running.
- Volume must be frozen before format.
- Sizing effect: rounded to multiples of `NR_OF_HDDS_PER_NODE * NR_OF_NODES * 256 GiB`.

## Procedure

1. Freeze volume:

```shell
csvol -E -v <VOLUME_ID> -t all
```

2. Format with 256 GiB partition size:

```shell
SDFS_PARTITION_SIZE_BYTES=$((256*1024*1024*1024)) sdfs format <VOLUME_ID>
```

3. Enable search optimization label:

```shell
cslabel -v <VOLUME_ID> -a -l usesearchopt
```

4. Enable init optimization label:

```shell
cslabel -v <VOLUME_ID> -a -l useinitopt
```

## Notes

- If 256 GiB partition sizing is unsuitable, use smaller partition settings.
- Internal recommendation based on legacy performance findings.

## De-duplication note

Canonical SDFS and backup operations references:

- `documents/cos/sdfs-overview-and-volumes.md`
- `documents/cos/confd-backup-and-restore.md`

## Reference

- <https://exasol.atlassian.net/browse/EXASOL-2034>


