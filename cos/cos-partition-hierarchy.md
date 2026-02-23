---
tool_name: cos
doc_type: concept
category: COS Partitions
title: "COS Partition Hierarchy"
summary: "COS organizes partitions in a hierarchical tree structure, similar to a process tree in Unix/Linux systems."
---
# COS Partition Hierarchy


## Partition Tree Structure

COS organizes partitions in a hierarchical tree structure, similar to a process tree in Unix/Linux systems.

### Complete Hierarchy Example

```
Root Node (Node 0)
├── System Services (Parent: 0)
│   ├── sshd (ID: 16)
│   ├── cron (ID: 17)
│   ├── logd (ID: 18)
│   ├── lockd (ID: 19)
│   ├── bucketfsd (ID: 25)
│   ├── healthd (ID: 26)
│   ├── dwad (ID: 27)
│   ├── cos_storage (ID: 30)
│   ├── confd (ID: 31)
│   ├── db-restapi (ID: 32)
│   └── eventd (ID: 42)
│
└── Database Cluster (Parent: 27 - dwad)
    └── controller-Exasol (ID: 56832)
        ├── pddserver (ID: 56833)
        ├── objectserver (ID: 56834)
        ├── exasqllog (ID: 56836)
        ├── loaderd (ID: 56837)
        ├── exaetl (ID: 56838)
        ├── exacs (ID: 56839)
        └── exasql (ID: 56840)
```

## Partition Properties

Each partition has specific attributes that define its behavior and relationships.

### Property Reference

| Property | Description | Example |
|----------|-------------|---------|
| **ID** | Unique partition identifier | 56832 |
| **OWNER** | User ID running the process | 500 (exadefusr) |
| **GROUP** | Group ID of the process | 500 (exausers) |
| **PARENT** | Parent partition ID | 27 (dwad) |
| **FLAGS** | Partition behavior flags | RAEI, --NI |
| **ONLINE NODES** | Active nodes running partition | 1/1, 3/4 |
| **COMMAND** | Executable running in partition | controller-Exasol |

### Viewing Partition Properties

```bash
# Basic partition list
cosps

# Example output:
ID    OWNER GROUP PARENT FLAGS ONLINE NODES   PHYSICAL NODES  COMMAND
16    0     0     0      R--I  11              -               sshd
27    0     0     0      RA-I  11              -               dwad
56832 500   500   27     --EK  11              -               controller-Exasol
```

```bash
# Show with names instead of IDs
cosps -m

# Show full paths and details
cosps -f

# Show as tree
cospstree
```

## Partition Flags

Flags control partition behavior and lifecycle.

### Flag Reference

| Flag | Name | Description |
|------|------|-------------|
| **R** | Restart | Partition automatically restarts if it fails |
| **A** | Active | Partition is currently active |
| **E** | External | Can be accessed from outside COS |
| **I** | Internal | Internal COS service |
| **K** | Keep | Preserve partition after process exit |
| **N** | No restart | Don't restart on failure |
| **S** | Suspended | Partition created but not started |
| **-** | Not set | Flag not enabled |

### Common Flag Combinations

#### RAEI - System Service
```
Restart + Active + External + Internal
```
- **Example**: `sshd`, `bucketfsd`, `confd`
- **Behavior**: Auto-restarts on failure, externally accessible
- **Use Case**: Critical system services

#### RA-I - Internal Service
```
Restart + Active + Internal
```
- **Example**: `dwad`, `logd`, `cos_storage`
- **Behavior**: Auto-restarts, not directly accessible externally
- **Use Case**: Core infrastructure services

#### --EK - Database Controller
```
External + Keep
```
- **Example**: `controller-Exasol`
- **Behavior**: No auto-restart (managed by parent), persistent, externally accessible
- **Use Case**: Database entry point controlled by DWAD

#### --NI - Database Component
```
No restart + Internal
```
- **Example**: `exasql`, `pddserver`, `objectserver`
- **Behavior**: No auto-restart (managed by controller), internal only
- **Use Case**: Database sub-components managed by controller

#### R--I - SSH Service
```
Restart + Internal
```
- **Example**: `sshd`
- **Behavior**: Auto-restarts, allows external access
- **Use Case**: Administrative access

## Parent-Child Relationships

### Relationship Rules

1. **Parent controls child lifecycle**: Killing parent terminates all children
2. **Children inherit restrictions**: Resource limits flow down
3. **Startup order**: Parent starts before children
4. **Communication**: Children can access parent context
5. **Dependency**: Children cannot exist without parent

### Viewing Relationships

```bash
# Show all children of DWAD (ID: 27)
cosps -p 27

# Show children of database controller (ID: 56832)
cosps -p 56832

# Show full tree from specific partition
cospstree 27

# Find parent of specific partition
cosps | awk '/^56840/ {print "Parent ID:", $4}'
```

## System Service Layer (Parent: 0)

### Root-Level Services

All system services have **Parent ID: 0**, meaning they're top-level partitions.

```bash
ID    OWNER GROUP PARENT FLAGS COMMAND
16    0     0     0      R--I  sshd
17    0     0     0      RA-I  cron
18    0     0     0      RA-I  logd
19    0     0     0      RA-I  lockd
25    500   500   0      RAEI  bucketfsd-bfsdefault
26    0     0     0      RA-I  healthd
27    0     0     0      RA-I  dwad
30    0     0     0      RA-I  cos_storage
31    0     0     0      RA-I  confd
32    0     0     0      RANI  db-restapi
42    0     0     0      RA-I  eventd
```

**Characteristics**:
- Started by COS init system
- Most run as root (UID 0)
- Auto-restart enabled (R flag)
- Independent of each other (except DWAD controls databases)

## Database Layer (Parent: 27)

### DWAD as Database Parent

DWAD (Database Watchdog Daemon) is the parent for all database instances:

```bash
ID    OWNER GROUP PARENT FLAGS COMMAND
27    0     0     0      RA-I  dwad
56832 500   500   27     --EK  controller-Exasol
```

**Why DWAD is separate**:
- Isolates database lifecycle from other services
- Enables database-specific restart policies
- Provides single control point for all databases
- Manages multi-database coordination

### Controller as Component Parent

Each database controller is the parent for its components:

```bash
ID    OWNER GROUP PARENT FLAGS COMMAND
56832 500   500   27     --EK  controller-Exasol
56833 500   500   56832  --NI  pddserver-Exasol
56834 500   500   56832  --NI  objectserver-Exasol
56836 500   500   56832  --NI  exasqllog-Exasol
56837 500   500   56832  --NI  loaderd-Exasol
56838 500   500   56832  --NI  exaetl-1024-Exasol
56839 500   500   56832  --NI  exacs-Exasol
56840 500   500   56832  --NI  exasql-7-Exasol
```

**Controller responsibilities**:
- Start database components in correct order
- Allocate resources to children
- Handle component failures
- Coordinate multi-node operation
- Provide database entry point

## Partition Lifecycle

### Startup Sequence

1. **COS Init**: Node starts, COS initializes
2. **Root services**: Parent 0 partitions start (sshd, confd, dwad, etc.)
3. **DWAD activation**: DWAD loads cluster metadata
4. **Database request**: Administrator requests database start
5. **Controller creation**: DWAD creates controller partition
6. **Component startup**: Controller creates child partitions (pddserver, exasql, etc.)
7. **Database ready**: All components online, database accepts connections

### Shutdown Sequence

1. **Database stop requested**: Via `confd_client` or `dwad_client`
2. **Controller signals children**: Graceful shutdown signal sent
3. **Components terminate**: Each child process exits cleanly
4. **Controller exits**: After all children stopped
5. **DWAD cleanup**: Removes controller partition
6. **Database offline**: Resources released

### Restart Behavior

**Flag-based restart**:
```bash
# R flag set: Auto-restart on failure
cosps | grep "RA"  # Services with restart enabled

# No R flag: Manual restart required
cosps | grep " --"  # No auto-restart (database components)
```

**Parent-controlled restart**:
- DWAD restarts database controller if needed
- Controller restarts components if needed
- System services restart themselves (R flag)

## Multi-Node Partitions

### Node Distribution

Partitions can span multiple physical nodes:

```bash
# Single-node partition
ONLINE NODES: 1/1  # Running on 1 of 1 configured nodes

# Multi-node partition
ONLINE NODES: 3/4  # Running on 3 of 4 configured nodes
```

### Viewing Node Details

```bash
# Show physical node IDs
cosps -N

# Show which nodes are offline for a partition
cosps -r

# Show root nodes
cosps -n
```

### Node Management

```bash
# Add node to partition
cosadd -N n12 56832

# Remove node from partition
cosrm -n 12 56832

# Move partition to different node
cosmv -N n12 56832
```

## Partition Context

### Execution Context

Each partition provides isolated execution context:

```bash
# Execute in partition context
cosexec 56832 -- pwd
# Output: /

cosexec 56832 -- id
# Output: uid=500(exadefusr) gid=500(exausers)

cosexec 56832 -- env | grep EXA
# Shows partition-specific environment variables
```

### Context Inheritance

Children inherit context from parent:
- Environment variables (can add/override)
- Resource limits (cannot exceed parent)
- User/group (can be different)
- Network namespace (typically shared)

## Finding Partitions

### By Command Name

```bash
# Find database controller
cosps | grep controller

# Find all database components
cosps | grep Exasol

# Find BucketFS
cosps | grep bucketfs
```

### By User

```bash
# Show partitions owned by exadefusr (UID 500)
cosps -u 500

# Show partitions owned by root
cosps -u 0
```

### By Parent

```bash
# Show all databases (children of DWAD)
cosps -p 27

# Show all components of specific database
cosps -p 56832
```

### By Flags

```bash
# Find all auto-restarting services
cosps | awk '$5 ~ /R/ {print}'

# Find external services
cosps | awk '$5 ~ /E/ {print}'
```

## Partition Tree Visualization

### Using cospstree

```bash
# Show full tree
cospstree

# Show subtree from specific partition
cospstree 27  # DWAD and all databases

# Show database components only
cospstree 56832  # Controller and children
```

### Tree Output Example

```
27 (dwad)
└── 56832 (controller-Exasol)
    ├── 56833 (pddserver-Exasol)
    ├── 56834 (objectserver-Exasol)
    ├── 56836 (exasqllog-Exasol)
    ├── 56837 (loaderd-Exasol)
    ├── 56838 (exaetl-1024-Exasol)
    ├── 56839 (exacs-Exasol)
    └── 56840 (exasql-7-Exasol)
```

## Best Practices


**Understand parent-child relationships**
- Never kill children directly
- Work through parent for lifecycle management
- Use `cospstree` to visualize dependencies


**Respect the hierarchy**
- DWAD → Controller → Components
- Killing DWAD stops all databases
- Killing controller stops all components


**Use flags to understand behavior**
- R flag = auto-restart
- E flag = externally accessible
- K flag = persistent partition


**Monitor the tree**
- Missing children indicate failures
- Orphaned partitions indicate issues
- Use `cosps -p` to check completeness

## Related Documentation

- [COS Overview](cos_overview.md)
- [COS System Partitions](cos_system_partitions.md)
- [COS Database Partitions](cos_database_partitions.md)
- [COS Commands Reference](cos_commands.md)
- [COS Troubleshooting](cos_troubleshooting.md)
- [COS Best Practices](cos_best_practices.md)
