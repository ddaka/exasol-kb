---
tool_name: confd_client
doc_type: reference
category: Node Management
subcommands: node_add, node_remove, node_list, node_info, node_state, node_resume, node_suspend, node_wait_offline, node_wait_ready, node_add_batch, node_clone_batch, node_ip_batch, nodes_terminate, master_change
---

# confd_client — Cluster Node Management

## Overview

Commands for managing cluster nodes: adding, removing, suspending, resuming, and batch operations on physical cluster infrastructure.

All commands run inside the COS namespace (SSH port 20002).

## node_add

This job adds a new **physical node** to the Exasol cluster (not a database reserve node).

**Use this command when**: Adding new hardware to the cluster infrastructure

**Don't use this for**: Adding reserve nodes to a database (use `db_add_reserve_nodes`)

Example request (YAML format):
```yaml
```

**Permissions**: Groups: root, exaadm

**Parameters**:

- `priv_net` (str, required): IP address of the private network.
- `bg_rec_limit` (int, optional): Background recovery throughput limit in MiB/s.
- `name` (str, optional): Name of the desired node.
- `nid` (int, optional): Node ID of the desired node.
- `no_partition_extend` (bool, optional): Flag to block the extension of auto-add partitions from including this node.
- `offline` (bool, optional): Flag to add the node as offline instead of suspended.
- `pub_net` (str, optional): IP address of the public network.
- `space_warn_threshold` (int, optional): Threshold value in percent (0, 100) for a low storage space warning.

**Examples**:

```bash
confd_client node_add {priv_net: 10.10.10.1/24}
```

## node_remove

This job removes one or more nodes from the cluster. The node cannot be part
of an EXAStorage volume or a database unless the "force" flag is set. If the
node is in a suspended state, the job will fail.

**Permissions**: Groups: root, exaadm

**Parameters**:

- `nid` (int|list, required): Node ID of a single node, or a list of node IDs.
- `force` (bool, optional): When set, the node can be deleted even if it is part of an EXAStorage volume or a database.

**Examples**:

```bash
confd_client node_remove {nid: 12}
confd_client node_remove {nid: [12, 13]}
```

## node_list

This job lists all the nodes in the current configuration. Options with
empty values are omitted.

**Permissions**: Groups: root, exaadm

## node_info

This job shows extended node information for a single node.

**Permissions**: Groups: root, exaadm

**Parameters**:

- `nid` (int, required): Node ID of the node.

**Examples**:

```bash
confd_client node_info {nid: 13}
```

## node_state

This job shows the current state of all nodes.

**Permissions**: Groups: root, exaadm

## node_resume

This job will resume one or more suspended nodes.

**Permissions**: Users: root | Groups: root, exaadm

**Parameters**:

- `nid` (int|list, required): Node ID of a single node, or a list of node IDs.
- `enable` (bool, optional): 'Also set the nodes as enabled (default: False).'

**Examples**:

```bash
confd_client node_resume {nid: 12}
confd_client node_resume {nid: [12, 13]}
```

## node_suspend

This job suspends one or more nodes.

**Permissions**: Users: root | Groups: root, exaadm

**Parameters**:

- `nid` (int|list, required): Node ID of a single node, or a list of node IDs.
- `disable` (bool, optional): 'Also set the node as disabled in the configuration (default: False).'

**Examples**:

```bash
confd_client node_suspend {nid: 12}
confd_client node_suspend {nid: [12, 13]}
```

## node_wait_offline

This job waits for the given nodes to finish shutdown. It returns a list of
offline nodes (may be empty in case of errors).

**Permissions**: Groups: root, exaadm

**Parameters**:

- `nids` (list, required): List of node IDs that you want to wait for.
- `timeout` (int, optional): 'Time in sec. to wait for the given nodes (default: {cls.def_timeout}, min: {cls.min_timeout}).'

**Examples**:

```bash
confd_client node_wait_offline {nids: [12, 13]}
```

## node_wait_ready

This job waits for the given nodes to finish booting and be ready to accept
ConfD job requests. It returns a list of                 ready nodes (may be
empty in case of errors).

**Permissions**: Groups: root, exaadm

**Parameters**:

- `nids` (list, required): List of node IDs that you want to wait for.
- `timeout` (int, optional): 'Time in sec. to wait for the given nodes (default: {cls.def_timeout}, min: {cls.min_timeout}).'

**Examples**:

```bash
confd_client node_wait_ready {nids: [12, 13]}
```

## node_add_batch

This job adds new nodes that are defined in a list.

**Permissions**: Groups: root, exaadm

**Parameters**:

- `nodes` (list, required): Nodes with arguments.

**Examples**:

```bash
confd_client node_add_batch {nodes: [{name: n11, nid: 11, private_ip: 10.10.10.11, uuid: 7EAA8DEF9DF7483CABAD93774A4F130EBF2CE77F},
confd_client node_add_batch {name: n12, nid: 12, private_ip: 10.10.10.12, uuid: 39103D5FA93C46D3B8359918D01156ABD33B36EC}]}
```

## node_clone_batch

This job adds new nodes by cloning an existing node that is used as a
template.

**Permissions**: Groups: root, exaadm

**Parameters**:

- `nid` (int, required): Node ID for the existing node to be used as template.
- `num_nodes` (int, required): Number of nodes to be added.
- `disks` (list, optional): Node disks for the nodes that should be added.

**Examples**:

```bash
confd_client node_clone_batch {nid: 11, num_nodes: 3}
```

## node_ip_batch

This job changes the IP addresses in a list of nodes.

        private: 10.10.10.13}]}

**Permissions**: Groups: root, exaadm

**Parameters**:

- `nodes` (list, required): List of Node IDs for the nodes to change (with arguments).

**Examples**:

```bash
confd_client node_ip_batch {nodes: [{nid: 11, private: 10.10.10.11}, {nid: 12, private: 10.10.10.12}, {nid: 13,
```

## nodes_terminate

This job stops all Exasol processes on the given nodes. The nodes remain up
and running.

**Permissions**: Groups: root, exaadm

**Parameters**:

- `nids` (list, required): List of node IDs for the nodes that should be terminated.

**Examples**:

```bash
confd_client nodes_terminate {nids: [12, 13]}
```

## master_change

This job selects a new master node from a list of nodes. Do not execute
other jobs until this operation is done.

**Permissions**: Groups: root, exaadm

**Parameters**:

- `nids` (list, required): List of node IDs from which to select a master node.

**Examples**:

```bash
confd_client master_change {nids: [11, 12, 13]}
```
