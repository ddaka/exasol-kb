---
tool_name: c4
doc_type: guide
category: system
title: "How to change IP addresses of existing nodes in v8"
summary: "c4/EXAConf workflow to change node IP addresses without reinstall, including service restart and c4 config synchronization."
---
# How to change IP addresses of existing nodes in v8

## Purpose

Change node IP addresses on existing v8 clusters without full reinstall.

## Impact

Short downtime required because cluster services must be restarted.

## Preconditions

- New IPs are reachable and tested.
- Maintenance window approved.
- Access to COS and host c4 configs on all nodes.

## Procedure

1. Update node IPs in `/exa/etc/EXAConf` (`PrivateNet` / `PublicNet`) for each node.
2. Set `Checksum = COMMIT` and apply EXAConf commit.
3. Stop Exasol cloud service on each node:

```shell
systemctl stop c4_cloud_command
```

4. Update OS network config (for example netplan/sysconfig) and apply.
5. Reconnect SSH using new IPs.
6. Update c4 configs on all nodes:
   - rootless: `$INSTALL_DIR/.ccc/ccc/etc/c4.yaml`
   - rootful: `/var/lib/ccc/etc/c4.yaml`
   - adjust `CCC_HOST_ADDRS` and `CCC_PLAY_INSTANCES_IP_ADDRS`
7. Update `cluster.yaml` for local play (`~/.ccc/play/local/<ID>/conf/cluster.yaml`) on all nodes.
8. Verify with `c4 ps` that new IPs are shown.
9. Start services again:

```shell
systemctl --user start c4_cloud_command
# or
systemctl start c4_cloud_command
```

## Notes

- If a failed node should remain excluded after restart, set `State = disabled` for that node in EXAConf.
- Canonical node IP change command reference (alternative approach): `node_ip_batch`.

## De-duplication reference

- `documents/cos/confd-node-management.md` (`node_ip_batch`)


