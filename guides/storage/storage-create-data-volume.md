---
tool_name: confd_client
doc_type: reference
category: storage
subcommands:
  - st_volume_create
  - st_volume_info
  - st_volume_add_label
  - db_info
  - node_list
  - st_disk_list
---

# Create Data Volume

This document explains the step-by-step procedure for creating a new data volume in an Exasol database.

Database data in Exasol is stored on data volumes, which are assigned to storage disks. Most deployments have one primary data volume per database.

---

## Prerequisites

- Access to COS via c4
- Know the number of active nodes
- Disk space available
- Database owner information

---

## Procedure

### Step 1: Connect to COS

```bash
c4 connect -i <PLAY_ID> -s cos

# Example:
./c4 connect -i c3275f84 -s cos
```

**Find play ID:**
```bash
c4 ps
```

### Step 2: Determine Parameters

**Get database owner:**
```bash
confd_client db_info db_name: MY_DATABASE | grep owner -A 2

# Example output:
# owner:
# - 500
# - 500
```

**Get node information:**
```bash
confd_client node_list

# Count active nodes (not reserve nodes)
```

**Get disk name:**
```bash
confd_client st_disk_list

# Example output:
# - disk1
# - disk2
```

### Step 3: Create the Data Volume

```bash
confd_client st_volume_create \
  name: <VOLUME_NAME> \
  disk: <DISK_NAME> \
  type: data \
  size: '<SIZE> <UNIT>' \
  num_master_nodes: <NUM_NODES> \
  nodes: '[<NODE_IDS>]' \
  redundancy: <REDUNDANCY> \
  owner: '[<UID>, <GID>]'

# Example: 1 TiB data volume, 4 nodes, redundancy 2
confd_client st_volume_create \
  name: DataVolume1 \
  disk: disk1 \
  type: data \
  size: '1 TiB' \
  num_master_nodes: 4 \
  nodes: '[11, 12, 13, 14]' \
  redundancy: 2 \
  owner: '[500, 500]'
```

**Output:**
```
vid: 2
```
The command returns the **volume ID** (vid) of the new volume.

### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `name` | string | Name of the new volume |
| `disk` | string | Name of the disk to use |
| `type` | string | Must be `data` |
| `size` | string | Volume size with unit (e.g., '1 TiB', '500 GiB') |
| `num_master_nodes` | integer | Number of active nodes |
| `nodes` | list | List of node IDs (integers) |
| `redundancy` | integer | Redundancy level (typically 2) |
| `owner` | tuple/list | Database owner as [UID, GID] or [username, groupname] |

**Important notes:**
- Size is maximum capacity; actual usage grows dynamically
- `num_master_nodes` MUST match number of active nodes
- ConfD does internal rounding of size (check with `st_volume_info`)

### Step 4: Enable Optimizations (Recommended)

```bash
# Enable initialization optimization
confd_client st_volume_add_label \
  vname: DataVolume1 \
  label: useinitopt

# Enable search optimization
confd_client st_volume_add_label \
  vname: DataVolume1 \
  label: usesearchopt
```

**What these do:**
- `useinitopt`: Faster volume initialization
- `usesearchopt`: Optimized block search operations

---

## Verification

```bash
# Check volume info
confd_client st_volume_info vid: 2

# Or by name
confd_client st_volume_info vname: DataVolume1
```

**Example output:**
```
_sec_name: 'EXAVolume : DataVolume1'
block_size: 4194304
disk: disk1
labels:
- useinitopt
- usesearchopt
name: DataVolume1
nodes:
- 11
- 12
- 13
- 14
num_master_nodes: 4
owner:
- 500
- 500
permissions: 640
redundancy: 2
size: '1 TiB'
type: data
vid: 2
```

## Related Documentation

- [Storage Overview](./storage-overview.md)
- [Create Local Archive Volume](./storage-create-local-archive-volume.md)
- [Create Remote Archive Volume](./storage-create-remote-archive-volume.md)
- [Storage Capacity and Monitoring](./storage-capacity-and-monitoring.md)
