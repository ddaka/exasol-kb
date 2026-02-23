---
tool_name: confd_client
doc_type: reference
category: storage
subcommands:
  - st_volume_create
  - st_volume_info
  - st_volume_add_label
  - db_info
  - st_node_list
---

# Create Local Archive Volume

This document explains how to create a local archive volume for database backups stored within the Exasol cluster.

**Use cases:**
- Fast backup/restore (local storage)
- No external dependencies
- Lower network overhead

**Considerations:**
- Uses cluster storage capacity
- Subject to cluster failures
- Fixed size (manual expansion only)

---

## Prerequisites

- Access to COS via c4
- Know data volume configuration (disk, nodes, owner)
- Sufficient disk space for backups

---

## Procedure

### Step 1: Connect to COS

```bash
c4 connect -i <PLAY_ID> -s cos
```

### Step 2: Get Data Volume Information

Archive volume must match data volume configuration.

```bash
# Get data volume info
confd_client db_info db_name: MY_DATABASE

# Get storage node info
confd_client st_node_list
```

**Key information needed:**
- Disk name (same as data volume)
- Node IDs (same as data volume)
- Number of master nodes (same as data volume)
- Owner (same as database owner)

### Step 3: Create Local Archive Volume

```bash
confd_client st_volume_create \
  name: <ARCHIVE_VOLUME_NAME> \
  disk: <DISK_NAME> \
  type: archive \
  size: '<SIZE> <UNIT>' \
  num_master_nodes: <NUM_NODES> \
  nodes: '[<NODE_IDS>]' \
  redundancy: <REDUNDANCY> \
  block_size: '512 KiB' \
  stripe_size: '512 KiB' \
  partition_size: <PARTITION_SIZE> \
  shared: true \
  owner: '[<UID>, <GID>]'

# Example: 2 TiB local archive, 4 nodes, redundancy 2
confd_client st_volume_create \
  name: LocalArchiveVolume1 \
  disk: disk1 \
  type: archive \
  size: '2 TiB' \
  num_master_nodes: 4 \
  nodes: '[11, 12, 13, 14]' \
  redundancy: 2 \
  block_size: '512 KiB' \
  stripe_size: '512 KiB' \
  partition_size: 274877906944 \
  shared: true \
  owner: '[500, 500]'
```

**Output:**
```
vid: 3
```

### Archive-Specific Parameters

| Parameter | Type | Value | Description |
|-----------|------|-------|-------------|
| `block_size` | string | `'512 KiB'` | Fixed for archives |
| `stripe_size` | string | `'512 KiB'` | Fixed for archives |
| `partition_size` | integer | See table below | Depends on volume size |
| `shared` | boolean | `true` | Required for archives |

### Partition Size Table

| Volume Size | Partition Size |
|-------------|----------------|
| < 250 GiB | 4294967296 (4 GiB) |
| ≥ 250 GiB and < 1 TiB | 34359738368 (32 GiB) |
| ≥ 1 TiB | 274877906944 (256 GiB) |

### Step 4: Enable Optimizations

```bash
confd_client st_volume_add_label \
  vname: LocalArchiveVolume1 \
  label: useinitopt

confd_client st_volume_add_label \
  vname: LocalArchiveVolume1 \
  label: usesearchopt
```

---

## Verification

```bash
# Check archive volume info
confd_client st_volume_info vid: 3

# Or by name
confd_client st_volume_info vname: LocalArchiveVolume1
```

---

## Using the Archive Volume

**In BACKUP command:**
```sql
BACKUP DATABASE MY_DATABASE TO LOCAL ARCHIVE 'LocalArchiveVolume1';
```

## Related Documentation

- [Storage Overview](./storage-overview.md)
- [Create Data Volume](./storage-create-data-volume.md)
- [Create Remote Archive Volume](./storage-create-remote-archive-volume.md)
- [Storage Capacity and Monitoring](./storage-capacity-and-monitoring.md)
