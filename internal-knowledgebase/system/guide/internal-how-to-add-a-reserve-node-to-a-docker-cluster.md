---
tool_name: dwad_client
doc_type: guide
category: system
title: "Internal - Add a reserve node to a Docker cluster"
summary: "Legacy Docker-based workflow to add a new physical node and register it as database reserve node."
---
# Internal - Add a reserve node to a Docker cluster

## Purpose

Add a reserve node to an existing Docker-based Exasol cluster (example: `2+0` to `2+1`).

## Preconditions

- New Docker host has sufficient CPU/RAM/storage.
- File-backed device(s) prepared on the new host.
- Maintenance window approved.

## Procedure

1. Prepare node filesystem and template:

```shell
export CONTAINER_EXA="$HOME/container_exa/"
docker run -v "$CONTAINER_EXA":/exa --rm -i exasol/docker-db:<version> init-sc --template --num-nodes 3
truncate --size 10G "$HOME/container_exa/data/storage/dev.1"
```

2. On active node/container, add physical node to EXAConf:

```shell
docker exec -ti n11 /bin/bash
exaconf add-node /exa/etc/EXAConf -p <new_node_ip>/24 -n 13
# configure node disk mapping (manual EXAConf edit or exaconf add-node-disk)
```

3. Update DB node list and required storage settings in EXAConf (include new node ID).
4. Set checksum for commit:

```shell
sed -i '/Checksum =/c\ Checksum = COMMIT' /exa/etc/EXAConf
```

5. Restart containers to apply node configuration.
6. Insert reserve node into database:

```shell
dwad_client insert-rnode DB1 n13
```

7. Verify reserve-node assignment:

```shell
dwad_client sys-nodes DB1
```

8. Increase storage redundancy as needed and wait for recovery:

```shell
csresize -i -l2 -v0
```

9. Validate failover behavior. For legacy versions where segment auto-move is absent, use manual `csmove` if required.

## Notes

- If a failed node should remain out of startup quorum, set `State = disabled` in EXAConf for that node.
- For modern/non-legacy workflows prefer canonical ConfD operations where available.

## De-duplication note

Canonical command references:

- `documents/cos/confd-node-management.md`
- `documents/cos/confd-database-management.md` (`db_add_reserve_nodes`)
- `documents/cos/dwad-client.md` (`insert-rnode`)


