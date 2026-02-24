---
tool_name: cos
doc_type: guide
category: system
title: "Internal - Enlarge file device on Docker"
summary: "Increase Exasol Docker file-device size by extending backing files and enlarging devices with cshdd."
---
# Internal - Enlarge file device on Docker

## Purpose

Increase storage capacity of file-backed devices in Exasol Docker deployments.

## Preconditions

- Sufficient free space on host storage used by Docker.
- Planned maintenance window.

## Procedure

1. Enter Exasol Docker container:

```shell
docker exec -ti exasoldb /bin/bash
```

2. Stop database:

```shell
dwad_client stop-wait DB
```

3. Extend file-backed device size (example +10GB):

```shell
truncate --size=+10GB /exa/data/storage/dev.1.data
# or newer layout:
truncate --size=+10GB /exa/data/storage/dev.1
```

4. Enlarge EXAStorage device:

```shell
cshdd --enlarge -n 11 -h /exa/data/storage/dev.1.data
# or newer layout:
cshdd --enlarge -n 11 -h /exa/data/storage/dev.1
```

5. Repeat `truncate` + `cshdd --enlarge` for all relevant device files/nodes.
6. Verify new device sizes:

```shell
csinfo -D
```

7. Start database:

```shell
dwad_client start DB
```

## LVM note

If Docker storage uses LVM, ensure PV/LV are extended first, for example:

- `pvresize /dev/<disk>`
- `lvextend -r -L +10GB /dev/<VG>/<LV>`
- Expand partition table as needed.

## De-duplication note

Canonical storage command references:

- `documents/cos/cos-storage-commands.md`


