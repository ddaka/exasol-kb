---
tool_name: confd_client
doc_type: guide
category: system
title: "Scenario: switch active node to reserve node within one site"
summary: "Maintenance workflow to suspend an active DB node, move storage ownership, and reintroduce the node as reserve."
---
# Scenario: switch active node to reserve node within one site

## Purpose

Temporarily remove an active node for maintenance and keep database availability by using a reserve node.

## Impact

Database restart required. Increased load/IO expected during restore/rebalance.

## Procedure

1. Identify active/reserve DB nodes:

```shell
confd_client db_info db_name: <DB_NAME>
```

2. Identify free target storage node per data/archive volume:

```shell
confd_client st_volume_info vname: <VOLUME_NAME>
```

3. Suspend node from DB workload:

```shell
confd_client db_stop db_name: <DB_NAME>
confd_client db_suspend_nodes db_name: <DB_NAME> node_list: [<NODE_ID>]
confd_client db_start db_name: <DB_NAME>
```

4. Move storage segments for each affected volume:

```shell
confd_client st_volume_move_node src_nodes: [<OLD_NODE>] dst_nodes: [<NEW_NODE>] vname: <VOLUME_NAME>
```

5. Suspend node in storage and cluster:

```shell
confd_client st_node_suspend nid: <NODE_ID>
confd_client node_suspend nid: <NODE_ID> disable: true
```

6. Stop services on deactivated node (if needed):

```shell
systemctl --user stop c4
systemctl --user stop c4_cloud_command
```

7. After maintenance, resume cluster/storage node and services:

```shell
confd_client node_resume nid: <NODE_ID> enable: true
systemctl --user start c4
systemctl --user start c4_cloud_command
confd_client st_node_resume nid: <NODE_ID>
```

8. Add node back as DB reserve node:

```shell
confd_client db_add_reserve_nodes db_name: <DB_NAME> node_list: [<NODE_ID>]
```

9. Verify node states:

```shell
confd_client node_state
confd_client db_info db_name: <DB_NAME>
```

## Notes

- Prefer local data access (DB node aligned with storage placement) to avoid performance degradation.
- If `disable=false` during suspend, cluster restarts may wait for that node.

## De-duplication note

Canonical command references:

- `documents/cos/confd-database-management.md`
- `documents/cos/confd-node-management.md`
- `documents/cos/confd-storage-devices.md`


