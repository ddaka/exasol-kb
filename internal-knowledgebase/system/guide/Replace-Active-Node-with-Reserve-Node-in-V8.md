---
tool_name: confd_client
doc_type: guide
category: system
title: "Replace active node with reserve node in v8"
summary: "Swap an active database node with a reserve node for maintenance and rebalance storage segments when required."
---
# Replace active node with reserve node in v8

## Purpose

Perform controlled active-to-reserve node swap for maintenance in Exasol v8.

## Prerequisites

- Cluster has at least one reserve node.
- Maintenance window approved.
- Node/volume IDs for target database are known.

## Procedure

### 1. Stop database

```shell
confd_client db_stop db_name: <db_name>
```

### 2. Suspend active node

```shell
confd_client db_suspend_nodes db_name: <db_name> node_list: '[n12]'
```

### 3. Start database

```shell
confd_client db_start db_name: <db_name>
```

### 4. Re-add old active node as reserve

```shell
confd_client db_add_reserve_nodes db_name: <db_name> node_list: '[n12]'
```

### 5. Check whether storage segment movement is required

```shell
confd_client db_info db_name: <db_name>
```

### 6. Move storage segments if needed

```shell
confd_client st_volume_move_node vname: <volume_name> src_nodes: '[12]' dst_nodes: '[13]'
```

Repeat for all relevant volumes (persistent and archive, if applicable).

## Validation

- Database is online.
- Node roles match expected active/reserve mapping.
- Storage segments are balanced and healthy.

## References

- <https://docs.exasol.com/db/latest/confd/jobs/db_stop.htm>
- <https://docs.exasol.com/db/latest/confd/jobs/db_start.htm>
- <https://docs.exasol.com/db/latest/confd/jobs/db_add_reserve_nodes.htm>
- <https://docs.exasol.com/db/latest/confd/jobs/db_suspend_nodes.htm>
- <https://docs.exasol.com/db/latest/confd/jobs/st_volume_move_node.htm>


