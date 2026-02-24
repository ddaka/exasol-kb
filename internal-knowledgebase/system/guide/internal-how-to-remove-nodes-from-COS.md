---
tool_name: confd_client
doc_type: guide
category: system
title: "How to remove nodes from COS and c4"
summary: "Operational sequence to remove database/storage nodes from cluster control plane and clean residual c4 metadata."
---
# How to remove nodes from COS and c4

## Purpose

Remove node(s) from database, storage, and COS cluster metadata, then clean c4 metadata if stale entries remain.

## Impact

- If node is active in database: downtime/restart required.
- If node only participates in storage metadata movement may be possible with reduced downtime.

## Procedure

1. Suspend active DB node(s):

```shell
confd_client db_stop db_name: <DB_NAME>
confd_client db_suspend_nodes db_name: <DB_NAME> node_list: [<NODE_ID>]
```

2. Remove reserve nodes from database (if applicable):

```shell
confd_client db_remove_reserve_nodes db_name: <DB_NAME> node_list: [<NODE_ID>]
```

3. Ensure node is not used by storage volumes. Move segments first if needed, then suspend storage node:

```shell
confd_client st_node_suspend nid: <NODE_ID>
```

4. Remove node from COS:

```shell
confd_client node_remove nid: [<NODE_ID>] force: true
```

5. Verify node removal:

```shell
cosps -N | grep <NODE_ID>
```

6. If `c4 ps` still shows removed nodes, clean c4 metadata on all nodes:

```shell
vim ~/.ccc/ccc/etc/c4.yaml
vim ~/.ccc/play/local/.../main/<NID>/conf/cluster.yaml
vim ~/.ccc/play/local/.../main/<NID>/updates/current.json
```

## De-duplication note

Canonical command syntax references:

- `documents/cos/confd-database-management.md`
- `documents/cos/confd-node-management.md`
- `documents/cos/confd-storage-devices.md`
- `documents/cos/confd-volume-management.md`


