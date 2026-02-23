---
tool_name: confd_client
doc_type: concept
category: storage
subcommands:
  - st_volume_create
  - st_volume_info
  - st_volume_list
  - remote_volume_add
  - remote_volume_list
---

# Exasol Storage Overview

This document describes the storage architecture, volume types, and core concepts in Exasol on-premise deployments.

## Storage Components

- **EXAStorage (SDFS)**: Exasol's distributed storage layer (Shared Distributed File System)
- **Volumes**: Logical storage containers for data and backups
- **Disks**: Physical or virtual storage devices underlying volumes
- **Redundancy**: Data replication for fault tolerance

## Management Tools

- **confd_client**: Volume and disk management commands (runs inside COS)
- **c4**: Infrastructure-level storage configuration (runs on host)
- **SQL**: Storage monitoring via system tables (EXA_VOLUME_SIZES, EXA_DB_SIZE_LAST_DAY)

---

## EXAStorage Architecture

**EXAStorage** (also called SDFS — Shared Distributed File System) is Exasol's distributed storage layer that provides:
- **High performance**: Optimized for database workloads
- **Fault tolerance**: Configurable redundancy levels
- **Scalability**: Add storage by adding nodes
- **Flexibility**: Supports various underlying storage technologies

### Storage Hierarchy

```
┌─────────────────────────────────────────────────────────┐
│                     Database                             │
│  ┌───────────────────────────────────────────────────┐  │
│  │                Data Volume                        │  │
│  │  ┌─────────────────────────────────────────────┐ │  │
│  │  │          EXAStorage (SDFS)                  │ │  │
│  │  │  ┌───────────────────────────────────────┐  │ │  │
│  │  │  │         Physical Disks                │  │ │  │
│  │  │  │  - SSD/NVMe/SAS                       │  │ │  │
│  │  │  │  - Block devices                      │  │ │  │
│  │  │  │  - LVM volumes                        │  │ │  │
│  │  │  │  - Instance store / EBS               │  │ │  │
│  │  │  └───────────────────────────────────────┘  │ │  │
│  │  └─────────────────────────────────────────────┘ │  │
│  └───────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│              Archive Volumes (Backups)                   │
│  ┌─────────────────────┐  ┌──────────────────────────┐  │
│  │  Local Archive      │  │  Remote Archive          │  │
│  │  (in cluster)       │  │  (S3/Azure/GCS/FTP)      │  │
│  └─────────────────────┘  └──────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

### Data Flow

**Write path:**
1. SQL INSERT/UPDATE → Database
2. Database → Data Volume
3. Data Volume → EXAStorage
4. EXAStorage → Physical disks (with redundancy)

**Backup path:**
1. BACKUP DATABASE command
2. Data read from Data Volume
3. Compressed (by default)
4. Written to Archive Volume (local or remote)

---

## Volume Types

Exasol uses three types of volumes:

### 1. Data Volumes (Persistent)

**Purpose**: Store persistent database data

**Characteristics:**
- Dynamic size (grows/shrinks automatically)
- Maximum size defined at creation
- Typically redundancy level 2
- Required for database operation

**Example:**
```bash
confd_client st_volume_create \
  name: DataVolume1 \
  type: data \
  size: '1 TiB' \
  redundancy: 2
```

### 2. Archive Volumes

**Purpose**: Store compressed database backups

**Two types:**
- **Local Archive Volumes**: Stored within Exasol cluster
- **Remote Archive Volumes**: Stored on external services (S3, Azure, FTP, etc.)

**Characteristics:**
- Fixed maximum size (manual expansion only for local)
- Typically redundancy level 2 (local only)
- Compressed by default (can be disabled)
- Automatic expiration of old backups

**Example:**
```bash
# Local archive
confd_client st_volume_create \
  name: LocalArchiveVolume1 \
  type: archive \
  size: '2 TiB' \
  redundancy: 2

# Remote archive (S3)
confd_client remote_volume_add \
  url: https://my-bucket.s3.eu-west-1.amazonaws.com \
  vol_type: s3 \
  remote_volume_name: RemoteArchiveVolume1
```

### 3. Temporary Data Volumes

**Purpose**: Store temporary tablespace data

**Characteristics:**
- **Automatically created** when database starts
- **Automatically deleted** when database stops
- Initial size: 1 GiB
- Dynamic growth
- Redundancy level 1 (no replication)

**No manual creation needed** — Exasol handles automatically.

---

## Redundancy Levels

**Redundancy** determines how many copies of data are stored:

| Level | Copies | Fault Tolerance | Use Case |
|-------|--------|-----------------|----------|
| **1** | 1 | None (no replication) | Temporary data only |
| **2** | 2 | Tolerates 1 disk/node failure | Standard production |
| **3** | 3 | Tolerates 2 disk/node failures | High-availability critical systems |

**Most common**: Redundancy 2 for data and archive volumes.

---

## Volume Nodes

**Master nodes**: The number of active nodes that will use the volume.

**Critical**: `num_master_nodes` must match the number of **active nodes** in the cluster.

**Example scenarios:**
```bash
# 4 active nodes + 1 reserve (4+1)
num_master_nodes: 4

# 8 active nodes + 2 reserve (8+2)
num_master_nodes: 8

# 3 active nodes, no reserve (3+0)
num_master_nodes: 3
```

**Why this matters**: Volumes are distributed across master nodes. Mismatch causes errors.

---

## Critical Paths

- Data: `/exa/data/storage/`
- BucketFS: `/exa/data/bucketfs/`
- Backups (staging): `/exa/data/s3backup/`
- Metadata: `/exa/metadata/storage/`

## Related Documentation

- [Storage Requirements](./storage-requirements.md)
- [Create Data Volume](./storage-create-data-volume.md)
- [Create Local Archive Volume](./storage-create-local-archive-volume.md)
- [Create Remote Archive Volume](./storage-create-remote-archive-volume.md)
- [Storage Capacity and Monitoring](./storage-capacity-and-monitoring.md)
