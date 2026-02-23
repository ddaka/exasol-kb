---
tool_name: cos
doc_type: concept
category: COS Partitions
title: "COS Database Partitions"
summary: "Database instances in COS run as a hierarchy of partitions under DWAD (ID: 27). Each database has a controller partition that manages multiple component partiti..."
---
# COS Database Partitions


## Overview

Database instances in COS run as a hierarchy of partitions under DWAD (ID: 27). Each database has a controller partition that manages multiple component partitions.

## Database Partition Hierarchy

```
dwad (ID: 27)
└── controller-Exasol (ID: 56832)
    ├── pddserver-Exasol (ID: 56833)
    ├── objectserver-Exasol (ID: 56834)
    ├── exasqllog-Exasol (ID: 56836)
    ├── loaderd-Exasol (ID: 56837)
    ├── exaetl-1024-Exasol (ID: 56838)
    ├── exacs-Exasol (ID: 56839)
    └── exasql-7-Exasol (ID: 56840)
```

## Controller Partition

### Controller Configuration

```bash
ID: 56832
OWNER: exadefusr (500)
GROUP: exausers (500)
PARENT: 27 (dwad)
FLAGS: --EK (External, Keep)
COMMAND: controller-Exasol
```

**Key Controller Parameters**:
```bash
-dbName Exasol                    # Database instance name
-port 8563                        # Client connection port
-mode rw                          # Read-write mode
-dbram 11184                      # Allocated RAM in MB (11.2 GB)
-commMain n11                     # Main communication node
-outputdir /exa/logs/db/Exasol/  # Log directory
```

### Controller Responsibilities

The controller is the master partition for a database instance:

1. **Lifecycle Management**
   - Start/stop database components
   - Manage component health
   - Handle graceful shutdown

2. **Resource Allocation**
   - Distribute RAM across components
   - Manage CPU allocation
   - Control disk I/O

3. **Client Interface**
   - Accept client connections (port 8563)
   - Route queries to SQL engine
   - Manage sessions

4. **Component Coordination**
   - Coordinate query execution
   - Manage transactions
   - Handle failover

### Controller Flags

**--EK (External, Keep)**:
- **External**: Accessible from outside COS (client connections)
- **Keep**: Partition persists after process exit
- **No Restart**: Managed by DWAD, not auto-restart
- **Not Internal**: Entry point for database access

### Viewing Controller

```bash
# Find database controller
cosps | grep controller

# Show controller details
cosps -f | grep controller-Exasol

# View controller's children
cosps -p 56832

# Check controller environment
cosps -e -p 56832
```

## Database Components

### Component Overview

Each database component runs as a child partition of the controller:

| Component | ID | Process Nr | Purpose |
|-----------|-----|------------|---------|
| **pddserver** | 56833 | 1 | Process Distribution Daemon |
| **objectserver** | 56834 | 2 | Metadata and Catalog |
| **exasqllog** | 56836 | 4 | Audit Logging |
| **loaderd** | 56837 | 5 | IMPORT/EXPORT Operations |
| **exaetl** | 56838 | 1024 | ETL Transformations |
| **exacs** | 56839 | 3 | In-Memory Cache |
| **exasql** | 56840 | 7 | SQL Query Engine |

### Component Details

All components share these properties:
```bash
OWNER: exadefusr (500)
GROUP: exausers (500)
PARENT: 56832 (controller)
FLAGS: --NI (No restart, Internal)
```

---

### 1. PDD Server (Process Distribution Daemon) - ID: 56833

**Purpose**: Coordinates query execution across cluster nodes

**Process Nr**: 1

**Responsibilities**:
- Query plan distribution
- Multi-node coordination
- Task scheduling
- Result aggregation
- Load balancing

**When it's used**:
- Every SQL query execution
- Distributed joins
- Parallel operations
- Multi-node aggregations

**Checking status**:
```bash
# Check if running
cosps | grep pddserver

# View logs
cosexec 56832 -- tail -f /exa/logs/db/Exasol/pddserver-Exasol-*.log

# Check process
cosexec 56833 -- ps aux | grep pdd
```

---

### 2. Object Server - ID: 56834

**Purpose**: Manages database metadata and system catalog

**Process Nr**: 2

**Responsibilities**:
- Table/schema definitions
- User permissions
- View definitions
- Function metadata
- Statistics storage
- System tables (EXA_*)

**Data managed**:
- `EXA_ALL_TABLES`
- `EXA_ALL_COLUMNS`
- `EXA_ALL_USERS`
- `EXA_ALL_SCHEMAS`
- Database objects catalog

**Checking status**:
```bash
# Check if running
cosps | grep objectserver

# View logs
cosexec 56832 -- tail -f /exa/logs/db/Exasol/objectserver-Exasol-*.log

# Query system tables (requires database running)
exaplus -c hostname:8563 -u sys -p exasol -sql "SELECT * FROM EXA_ALL_TABLES"
```

---

### 3. EXA SQL Log Server - ID: 56836

**Purpose**: Handles audit logging and query history

**Process Nr**: 4

**Responsibilities**:
- SQL audit trail
- Query history
- Performance statistics
- System events
- Statistical system tables

**Data logged**:
- Executed queries
- Query duration
- User activity
- Login/logout events
- System changes

**Log tables**:
- `EXA_DBA_AUDIT_SQL`
- `EXA_STATISTICS.EXA_SQL_LAST_DAY`
- `EXA_STATISTICS.EXA_USER_SESSIONS`

**Checking status**:
```bash
# Check if running
cosps | grep exasqllog

# View logs
cosexec 56832 -- tail -f /exa/logs/db/Exasol/exasqllog-Exasol-*.log

# Query audit log (requires database running)
exaplus -c hostname:8563 -u sys -p exasol -sql "SELECT * FROM EXA_DBA_AUDIT_SQL"
```

---

### 4. Loader Daemon - ID: 56837

**Purpose**: Manages IMPORT and EXPORT operations

**Process Nr**: 5

**Responsibilities**:
- Data import from external sources
- Data export to files/cloud
- Parallel loading
- File format handling (CSV, Parquet, Avro, ORC)
- S3/Azure/GCS connectivity

**Used for**:
- `IMPORT` statements
- `EXPORT` statements
- Bulk data loading
- ETL data movement
- Cloud storage integration

**Checking status**:
```bash
# Check if running
cosps | grep loaderd

# View logs
cosexec 56832 -- tail -f /exa/logs/db/Exasol/loaderd-Exasol-*.log

# Monitor during import/export
coswatch 56837 &
```

---

### 5. ETL Daemon - ID: 56838

**Purpose**: Handles ETL transformations and processing

**Process Nr**: 1024

**Responsibilities**:
- Complex data transformations
- ETL job execution
- Data cleansing operations
- Integration with ETL tools

**Checking status**:
```bash
# Check if running
cosps | grep exaetl

# View logs
cosexec 56832 -- tail -f /exa/logs/db/Exasol/exaetl-Exasol-*.log
```

---

### 6. EXA Cache Server - ID: 56839

**Purpose**: Manages in-memory data caching

**Process Nr**: 3

**Responsibilities**:
- Hot data caching
- Query result caching
- Memory management
- Cache invalidation
- Performance optimization

**What's cached**:
- Frequently accessed tables
- Query results
- Intermediate results
- Metadata lookups

**Checking status**:
```bash
# Check if running
cosps | grep exacs

# View logs
cosexec 56832 -- tail -f /exa/logs/db/Exasol/exacs-Exasol-*.log

# Monitor memory usage
cosexec 56839 -- ps aux | grep exacs
```

---

### 7. EXA SQL Server - ID: 56840

**Purpose**: SQL query execution engine (the core)

**Process Nr**: 7

**Responsibilities**:
- SQL parsing
- Query optimization
- Query execution
- Transaction management
- Data access
- Result generation

**This is the heart**:
- Executes all SQL queries
- Manages transactions
- Enforces constraints
- Performs calculations
- Accesses storage

**Checking status**:
```bash
# Check if running
cosps | grep exasql-7

# View logs
cosexec 56832 -- tail -f /exa/logs/db/Exasol/exasql-Exasol-*.log

# Monitor CPU usage
cosexec 56840 -- top -b -n 1 | grep exasql

# Check connections
exaplus -c hostname:8563 -u sys -p exasol -sql "SELECT * FROM EXA_ALL_SESSIONS"
```

## Component Flags: --NI

All database components have **--NI** flags:

**--NI (No restart, Internal)**:
- **No Restart**: Controller manages lifecycle, no auto-restart
- **Internal**: Not directly accessible from outside
- Components are started/stopped as a group by controller
- Failures trigger controller-level recovery

### Why No Auto-Restart?

Components must start in specific order and with coordination:
1. Controller allocates resources
2. Controller starts components sequentially
3. Components must all be ready before database accepts connections
4. Stopping one component requires stopping all

**If component fails**:
- Controller detects failure
- Controller stops all components
- Controller restarts entire database
- Ensures consistent state

## Database Lifecycle

### Starting a Database

```bash
# Via confd_client (recommended)
confd_client -c db_start -a "db: Exasol"
```

**What happens**:
1. DWAD receives start request
2. DWAD creates controller partition (56832)
3. Controller allocates resources
4. Controller starts components in order:
   - pddserver (coordination)
   - objectserver (metadata)
   - exasqllog (logging)
   - loaderd (import/export)
   - exaetl (ETL)
   - exacs (cache)
   - exasql (SQL engine)
5. Database becomes available on port 8563

### Stopping a Database

```bash
# Via confd_client (recommended)
confd_client -c db_stop -a "db: Exasol"
```

**What happens**:
1. DWAD receives stop request
2. DWAD signals controller to shutdown
3. Controller stops accepting new connections
4. Controller completes in-flight transactions
5. Controller stops components in reverse order
6. Controller exits
7. DWAD removes partition

### Database State

```bash
# Check database state
confd_client -c db_state -a "db: Exasol" -j

# Possible states:
# - running: All components online
# - stopped: No controller partition
# - starting: Controller exists, components starting
# - stopping: Shutdown in progress
# - crashed: Unexpected termination
```

## Monitoring Database Partitions

### Check All Components

```bash
# View database and all components
cosps -p 27  # DWAD's children (all databases)

# View specific database components
cosps -p 56832  # Controller's children

# Should show 7 child partitions
```

### Expected Output

```bash
ID    OWNER GROUP PARENT FLAGS ONLINE NODES COMMAND
56832 500   500   27     --EK  1/1           controller-Exasol
56833 500   500   56832  --NI  1/1           pddserver-Exasol
56834 500   500   56832  --NI  1/1           objectserver-Exasol
56836 500   500   56832  --NI  1/1           exasqllog-Exasol
56837 500   500   56832  --NI  1/1           loaderd-Exasol
56838 500   500   56832  --NI  1/1           exaetl-1024-Exasol
56839 500   500   56832  --NI  1/1           exacs-Exasol
56840 500   500   56832  --NI  1/1           exasql-7-Exasol
```

### Missing Components

If components are missing, database is not fully started:

```bash
# Count components
cosps -p 56832 | wc -l
# Should be 8 (7 components + header)

# Find missing
echo "Expected: pddserver, objectserver, exasqllog, loaderd, exaetl, exacs, exasql"
cosps -p 56832
```

## Accessing Database Partitions

### Execute in Controller Context

```bash
# View controller's logs
cosexec 56832 -- ls -la /exa/logs/db/Exasol/

# Check database configuration
cosexec 56832 -- cat /exa/etc/database_Exasol.conf

# View environment
cosexec 56832 -- env | grep EXA
```

### Execute in Component Context

```bash
# Check SQL engine process
cosexec 56840 -- ps aux | grep exasql

# View cache server memory
cosexec 56839 -- ps aux | grep exacs

# Check PDD server connections
cosexec 56833 -- netstat -an | grep ESTABLISHED
```

## Multi-Node Database Partitions

In multi-node clusters, database partitions span nodes:

```bash
# Example: 4-node cluster
ONLINE NODES: 4/4  # All nodes running partition

# Degraded: One node offline
ONLINE NODES: 3/4  # Still operational

# Critical: Too many nodes offline
ONLINE NODES: 1/4  # May not be functional
```

### Viewing Node Distribution

```bash
# Show physical nodes for each partition
cosps -N -p 56832

# Show offline nodes
cosps -r -p 56832
```

## Database Resource Allocation

### Memory (dbram)

Controller parameter specifies total database RAM:

```bash
-dbram 11184  # 11184 MB = ~11.2 GB
```

**Distribution** (approximate):
- exasql: ~70% (query execution)
- exacs: ~15% (cache)
- objectserver: ~5% (metadata)
- Other components: ~10%

### CPU

Components share CPU resources:
- exasql: Most CPU-intensive (queries)
- pddserver: Coordination overhead
- Others: Minimal CPU usage

## Troubleshooting Database Partitions

### Database Won't Start

**Check DWAD**:
```bash
cosps | grep dwad
# Should be running (ID: 27)
```

**Check for controller**:
```bash
cosps | grep controller-Exasol
# Should exist after start request
```

**Check logs**:
```bash
# DWAD logs
cat /exa/logs/cored/confD.log | grep Exasol

# Controller logs (if exists)
cosexec 56832 -- cat /exa/logs/db/Exasol/cored_Exasol_*.log
```

### Component Missing

**Check all components**:
```bash
cosps -p 56832 | wc -l
# Should be 8 lines (including header)
```

**View controller logs**:
```bash
cosexec 56832 -- tail -100 /exa/logs/db/Exasol/cored_Exasol_*.log
```

**Restart database**:
```bash
confd_client -c db_stop -a "db: Exasol"
sleep 5
confd_client -c db_start -a "db: Exasol"
```

### Component Crashed

**Never kill components directly!**

```bash
# ❌ DON'T DO THIS
coskill 56840  # Don't kill exasql directly

# ✅ DO THIS
confd_client -c db_restart -a "db: Exasol"

# OR kill controller (DWAD will restart)
coskill 56832  # Controller manages all children
```

## Best Practices


**Use confd_client for database operations**
```bash
confd_client -c db_start -a "db: Exasol"
confd_client -c db_stop -a "db: Exasol"
confd_client -c db_restart -a "db: Exasol"
```


**Never kill components directly**
- Always work through controller
- Components must stop together
- Maintains database consistency


**Monitor all components**
```bash
# Should always be 7 components
cosps -p 56832 | tail -n +2 | wc -l
```


**Check logs for issues**
```bash
# Controller logs are most important
cosexec 56832 -- tail -f /exa/logs/db/Exasol/cored_Exasol_*.log
```

## Related Documentation

- [COS Overview](cos_overview.md)
- [COS Partition Hierarchy](cos_partition_hierarchy.md)
- [COS System Partitions](cos_system_partitions.md)
- [COS Commands Reference](cos_commands.md)
- [COS Troubleshooting](cos_troubleshooting.md)
- [COS Best Practices](cos_best_practices.md)
