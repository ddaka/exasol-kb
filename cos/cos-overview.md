---
tool_name: cos
doc_type: concept
category: COS Architecture
title: "COS Overview and Architecture"
summary: "The Cluster Operating System (COS) in Exasol is a sophisticated container orchestration layer that manages database processes through a hierarchical partition s..."
---
# COS Overview and Architecture


## What is COS?

The Cluster Operating System (COS) in Exasol is a sophisticated container orchestration layer that manages database processes through a hierarchical partition system. Each service, daemon, and database component runs in isolated partitions with controlled resource allocation and process relationships.

## Core Concepts

### Partitions as Containers

COS partitions are lightweight, isolated execution environments similar to containers or namespaces in modern container orchestration systems.

**Partitions provide:**
- **Process Isolation**: Each partition runs specific processes with defined ownership
- **Resource Control**: Memory, CPU, and network resources allocated per partition
- **Hierarchical Structure**: Parent-child relationships enable process group management
- **Fault Isolation**: Process failures contained within partitions

### Container-like Architecture

COS predates modern container systems like Docker and Kubernetes but provides similar capabilities:

| Feature | COS Partitions | Docker/Kubernetes |
|---------|---------------|-------------------|
| **Process Isolation** | ✓ Partition namespaces | ✓ Container namespaces |
| **Resource Limits** | ✓ Memory, CPU quotas | ✓ cgroups limits |
| **Hierarchy** | ✓ Parent-child trees | ✓ Pod containers |
| **Auto-restart** | ✓ Flag-based | ✓ Restart policies |
| **Network** | ✓ Isolated/shared | ✓ Pod networks |
| **File System** | ✓ COS layer + /exa | ✓ Image layers |

## COS Versions

COS and database versions are independent:

```bash
EXA_OS_VERSION=8.53.2        # COS version
EXA_DB_VERSION=8.33.0        # Database version
EXA_IMG_VERSION=8.33.0       # Image version
EXA_RE_VERSION=8.8.0         # Runtime environment version
```

**Location**: `/opt/exasol/cos-8.53.2/`

## Why COS Exists

### Problem: Complex Database Cluster Management

Exasol databases consist of multiple interdependent processes:
- Multiple database instances
- Storage management layers
- Configuration services
- Backup systems
- Network services

### Solution: Partition-Based Orchestration

COS provides:
1. **Unified Management**: Single control plane for all processes
2. **Fault Recovery**: Automatic restart of failed components
3. **Resource Allocation**: Controlled memory and CPU distribution
4. **Multi-Node Coordination**: Synchronized operations across cluster
5. **Process Dependencies**: Parent-child relationships ensure correct startup order

## COS Components

### 1. Process Management Layer

- **cosps**: List and monitor partitions
- **cosexec**: Execute commands in partition context
- **coskill**: Terminate partitions
- **coswatch**: Real-time event monitoring

### 2. Storage Management Layer (CS)

- **csinfo**: Storage and volume information
- **csvol**: Volume management
- **cssnap**: Snapshot operations
- **csbench**: Performance testing

### 3. Configuration Management

- **ConfD**: Centralized configuration daemon
- **confd_client**: Configuration operations
- Cluster-wide synchronization

### 4. Database Lifecycle

- **DWAD**: Database watchdog daemon
- **dwad_client**: Database control operations
- Automatic failover and recovery

## File System Structure

```
/opt/exasol/                 # COS installation directory
├── cos-8.53.2/              # COS binaries and libraries
│   ├── bin/                 # COS commands (cosps, cosexec, etc.)
│   ├── lib/                 # COS libraries
│   └── share/doc/           # COS documentation
├── db-8.33.0/               # Database version
├── runtime-8.8.0-jdk/       # JDK runtime
└── slc-9.1.0/               # Script Language Container

/exa/                        # COS data directory
├── data/                    # Persistent data
│   ├── bucketfs/            # BucketFS storage
│   ├── storage/             # Database volumes
│   └── s3backup/            # Backup staging
├── etc/                     # Configuration files
│   ├── dwad/                # DWAD configuration
│   ├── remote_volumes/      # Remove volume mappings
│   └── ssl/                 # SSL certificates
├── logs/                    # Log files
│   ├── db/                  # Database logs
│   ├── logd/                # System logs
│   └── cored/               # Core dumps
└── metadata/                # Cluster metadata
    └── dwad/                # DWAD state
```

## COS in Multi-Node Clusters

### Single Node vs Multi-Node

**Single Node**:
- One physical node runs all partitions
- Simpler debugging and monitoring
- No distributed coordination overhead

**Multi-Node**:
- Partitions span multiple physical nodes
- Fault tolerance through redundancy
- DWAD coordinates partition distribution
- Failed nodes show offline partition count

### Node Identification

Each node has multiple identifiers:

```bash
# Physical node ID (persistent)
EXA_NODE_ID=11

# Hostname
HOSTNAME=n11

# Logical node ID (COS cluster)
cosinfo -n  # Returns logical node number
```

### Partition Distribution

```bash
# Check which nodes run a partition
cosps -f | grep "ONLINE NODES"

# Example output:
# ONLINE NODES: 3/4  # 3 of 4 nodes online for this partition
```

## COS Security Model

### User Isolation

Partitions run as specific users:

| User | UID | Purpose |
|------|-----|---------|
| **root** | 0 | System services (sshd, confd, dwad) |
| **exadefusr** | 500 | Database processes, BucketFS |
| **exausers** | 500 | Database group |

### Process Isolation

- Each partition has isolated process namespace
- Child partitions inherit parent restrictions
- Network isolation available (but typically shared for performance)
- File system access controlled by user permissions

### Access Control

```bash
# SSH access on custom port (not 22)
ssh -p 20002 root@node-hostname

# Partition access requires proper user
cosexec 56832 -- id
# Returns: uid=500(exadefusr) gid=500(exausers)
```

## Environment Variables

COS sets specific environment variables in each partition:

```bash
# Version information
EXA_OS_VERSION=8.53.2
EXA_DB_VERSION=8.33.0
EXA_IMG_VERSION=8.33.0
EXA_RE_VERSION=8.8.0

# Node identification
EXA_NODE_ID=11
HOSTNAME=n11

# Paths
COS_DIRECTORY=/opt/exasol/cos-8.53.2
CURRENTDB=/opt/exasol/db-8.33.0
CURRENTCOS=/opt/exasol/cos-8.53.2
CURRENTJDK=/opt/exasol/runtime-8.8.0-jdk
CURRENTSLC=/opt/exasol/slc-9.1.0-c4-6-standard-EXASOL-all

# Configuration
COS_CONF_DIR=/exa
TZ=Europe/Berlin

# BucketFS-specific (for user 500)
SDFS_PATH_FILE=/exa/etc/remote_volumes/exadefusr.500.conf
```

## COS vs Traditional Systems

### Comparison to SystemD

| Aspect | COS | SystemD |
|--------|-----|---------|
| **Scope** | Cluster-wide | Single machine |
| **Hierarchy** | Deep parent-child trees | Flat service units |
| **Restart Logic** | Parent controls children | Independent units |
| **Coordination** | Multi-node synchronization | Local only |
| **Resource Control** | Partition-level | Cgroup-based |

### Comparison to Kubernetes

| Aspect | COS | Kubernetes |
|--------|-----|-----------|
| **Architecture** | Monolithic cluster OS | Distributed orchestrator |
| **Workload Type** | Database-specific | General containers |
| **Isolation** | Lightweight partitions | Full containers |
| **Configuration** | ConfD centralized | Distributed etcd |
| **Networking** | Shared or isolated | Pod networks |
| **Storage** | CS integrated | External CSI |

## Key Takeaways


**COS is a complete cluster operating system**
- Not just process management
- Includes storage, configuration, monitoring


**Partitions are the fundamental unit**
- Similar to containers
- Hierarchical relationships
- Resource isolation


**DWAD manages all databases**
- Parent partition for database clusters
- Coordinates lifecycle operations


**Never kill child partitions directly**
- Always work through parent
- Maintain consistency


**COS commands are powerful**
- Deep inspection capabilities
- Multi-node coordination
- Rich debugging tools

## Getting Started

### Basic Commands

```bash
# List all partitions
cosps

# Show full information
cosps -f -m

# Execute command in partition
cosexec 56832 -- ls /exa

# View partition hierarchy
cospstree

# Monitor events
coswatch
```

### Finding Your Way

```bash
# Where am I?
cosinfo -p  # Current partition ID
cosinfo -n  # Current node ID
cosinfo -i  # Current hostname

# What's running?
cosps | grep -E "database|bucketfs|confd"

# Check cluster health
coscheck
```

## Related Documentation

- [COS Partition Hierarchy](cos_partition_hierarchy.md)
- [COS System Partitions](cos_system_partitions.md)
- [COS Database Partitions](cos_database_partitions.md)
- [COS Commands Reference](cos_commands.md)
- [COS Storage Commands](cos_storage_commands.md)
- [COS Troubleshooting](cos_troubleshooting.md)
- [COS Best Practices](cos_best_practices.md)
