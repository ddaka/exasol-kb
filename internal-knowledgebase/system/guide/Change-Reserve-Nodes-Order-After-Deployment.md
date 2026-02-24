---
tool_name: c4
doc_type: guide
category: system
title: "Change reserve node order after deployment"
summary: "Procedure to correct swapped reserve-node identity/order in Exasol v8 deployments by aligning EXAConf and c4 host mappings."
---
# Change reserve node order after deployment

## Purpose

Correct node identity/order mismatch after deployment (for example `n20` and `n21` swapped in host/IP mapping).

## When to use

- Node was replaced/reintroduced after initial deployment.
- Node ID, hostname, and IP order are inconsistent.
- Operational tooling shows swapped node identity.

## Preconditions

- Maintenance window.
- Full cluster access (COS + host OS).
- Database downtime allowed.
- Backup/recovery path validated.

## Procedure

### 1. Update EXAConf node definitions

In `/exa/etc/EXAConf`, swap affected node entries (`Node`, `Name`, `Affinity`, and related metadata) to target order.

Commit:

```shell
exaconf commit
```

### 2. Stop database

```shell
confd_client db_stop db_name: <db_name>
```

### 3. Stop c4 cloud command service on all hosts

```shell
sudo systemctl stop c4_cloud_command.service
```

### 4. Align `c4.yaml` host ordering on all hosts

Update deployment config:

- Rootless: `~/.ccc/ccc/etc/c4.yaml`
- Rootful: `/var/lib/ccc/etc/c4.yaml`

Ensure sorted/correct order for:

- `host.addrs`
- `host.external_addrs`
- `instances_ip_addrs`

### 5. Rename local c4 play node directories if required

On affected hosts, rename node folders under play directory so local metadata matches new node numbering.

### 6. Restart c4 cloud command service on all hosts

```shell
sudo systemctl start c4_cloud_command.service
```

### 7. Validate

- `c4 ps` shows corrected node identity/order.
- DB and storage nodes map to expected hostnames/IPs.
- Reserve role assignment is consistent.

## Post-step note

You may need to rebalance/move storage segments using `st_volume_move_node` after node-order correction.

## References

- <https://docs.exasol.com/db/latest/administration/on-premise/nodes/reserve_nodes/add_reserve_nodes_existing.htm>
- <https://docs.exasol.com/db/latest/confd/jobs/st_volume_move_node.htm>


