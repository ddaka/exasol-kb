---
tool_name: cos
doc_type: concept
category: COS Partitions
title: "COS System Partitions"
summary: "System partitions are root-level services (Parent ID: 0) that provide core infrastructure for the COS cluster. Most run as root and auto-restart on failure."
---
# COS System Partitions


## Overview

System partitions are root-level services (Parent ID: 0) that provide core infrastructure for the COS cluster. Most run as root and auto-restart on failure.

## System Partition List

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

## Core Services

### 1. SSH Daemon (sshd) - ID: 16

**Purpose**: Remote administrative access to COS namespace

**Configuration**:
```bash
ID: 16
OWNER: root (0)
GROUP: root (0)
PARENT: 0
FLAGS: R--I (Restart, Internal)
COMMAND: sshd -D -o PermitRootLogin=yes -p 20002
PORT: 20002 (NOT standard 22)
```

**Key Features**:
- Runs on custom port **20002**
- Root login enabled for admin access
- Separate from OS-level SSH
- Auto-restarts on failure (R flag)

**Accessing SSH**:
```bash
# Connect to COS namespace
ssh -p 20002 root@node-hostname

# Or from node OS
ssh -p 20002 root@localhost
```

**Common Operations**:
```bash
# Check if sshd is running
cosps | grep sshd

# View sshd logs
cosexec 16 -- journalctl -u sshd -n 50

# Restart sshd (automatic due to R flag)
coskill 16
```

---

### 2. DWAD - Database Watchdog (ID: 27)

**Purpose**: Database cluster lifecycle management

**Configuration**:
```bash
ID: 27
OWNER: root (0)
GROUP: root (0)
PARENT: 0
FLAGS: RA-I (Restart, Active, Internal)
COMMAND: dwad --backupfile /exa/metadata/dwad/dwad.dump
```

**Responsibilities**:
- Manages database instance startup and shutdown
- Monitors database health and availability
- Maintains cluster metadata
- Coordinates database failover
- Parent partition for all database instances

**DWAD is critical**:
- All databases are children of DWAD (Parent: 27)
- Killing DWAD stops ALL databases
- DWAD auto-restarts, but databases must be restarted manually

**Operations**:
```bash
# Check DWAD status
cosps | grep dwad

# View DWAD's children (all databases)
cosps -p 27

# Check DWAD metadata
ls -la /exa/metadata/dwad/

# Use dwad_client for operations
dwad_client --help
```

**Common Issues**:
- DWAD crash = all databases stop
- Corrupt metadata = databases won't start
- DWAD restart doesn't auto-start databases

---

### 3. BucketFS Daemon (bucketfsd) - ID: 25

**Purpose**: Distributed file system for scripts, drivers, and libraries

**Configuration**:
```bash
ID: 25
OWNER: exadefusr (500)
GROUP: exausers (500)
PARENT: 0
FLAGS: RAEI (Restart, Active, External, Internal)
COMMAND: bucketfsd-bfsdefault --config=/exa/etc/bucketfs.cfg_bfsdefault
ENVIRONMENT: SDFS_PATH_FILE=/exa/etc/remote_volumes/exadefusr.500.conf
```

**Storage Types**:
- **Local**: `/exa/data/bucketfs/`
- **Remote**: S3, Azure Blob, GCS (via SDFS)

**BucketFS Contents**:
- UDF (User-Defined Function) scripts
- JDBC drivers (Oracle, MySQL, PostgreSQL, etc.)
- Python/R libraries and packages
- Script Language Containers (SLC)
- Custom binaries and tools

**Accessing BucketFS**:
```bash
# From COS
cosexec 25 -- ls -la /exa/data/bucketfs/

# Via HTTP API (port 2580 by default)
curl http://node-hostname:2580/

# Check configuration
cat /exa/etc/bucketfs.cfg_bfsdefault

# Remote volumes (S3/Azure)
cat /exa/etc/remote_volumes/exadefusr.500.conf
```

**Operations**:
```bash
# Check BucketFS status
cosps | grep bucketfs

# View BucketFS logs
tail -f /exa/logs/cored/bucketfs.log

# Restart BucketFS
dwacli bucketfs restart bfsdefault

# Test connectivity
curl -u w:write http://localhost:2580/default/
```

---

### 4. ConfD - Configuration Daemon (ID: 31)

**Purpose**: Centralized cluster configuration management

**Configuration**:
```bash
ID: 31
OWNER: root (0)
GROUP: root (0)
PARENT: 0
FLAGS: RA-I (Restart, Active, Internal)
COMMAND: confd -v
```

**Responsibilities**:
- Manages cluster-wide configuration
- Synchronizes changes across all nodes
- Provides job-based API for operations
- Handles database lifecycle (start/stop/backup)
- Coordinates multi-node operations

**Using confd_client**:
```bash
# Get cluster information
confd_client -c info -j

# List available jobs
confd_client -c list

# Get help for specific job
confd_client -c help -a db_start

# Start database (async job)
confd_client -c db_start -a "db: Exasol" -S

# Check job status
confd_client -c result -a <job_id>

# Get database state
confd_client -c db_state -a "db: Exasol" -j
```

**Common Operations**:
```bash
# Database management
confd_client -c db_start -a "db: Exasol"
confd_client -c db_stop -a "db: Exasol"
confd_client -c db_state -a "db: Exasol"

# Backup operations
confd_client -c db_backup_start -a "db: Exasol"
confd_client -c db_backup_list

# Cluster operations
confd_client -c cluster_state
confd_client -c node_state
```

---

### 5. COS Storage (cos_storage) - ID: 30

**Purpose**: Manages COS storage layer and volumes

**Configuration**:
```bash
ID: 30
OWNER: root (0)
GROUP: root (0)
PARENT: 0
FLAGS: RA-I (Restart, Active, Internal)
COMMAND: cos_storage
```

**Responsibilities**:
- Volume management and allocation
- Disk I/O coordination
- Storage pool management
- Snapshot operations
- Data distribution across nodes

**Operations**:
```bash
# Check storage status
cosps | grep cos_storage

# View volumes
csinfo -v

# Show volume details
csinfo -V

# Check HDD information
csinfo -D

# View node storage
csinfo -n
```

---

### 6. Log Daemon (logd) - ID: 18

**Purpose**: Centralized logging service

**Configuration**:
```bash
ID: 18
OWNER: root (0)
GROUP: root (0)
PARENT: 0
FLAGS: RA-I (Restart, Active, Internal)
COMMAND: logd
```

**Log Locations**:
- `/exa/logs/logd/` - System logs
- `/exa/logs/db/<database>/` - Database logs
- `/exa/logs/cored/` - Core dumps

**Accessing Logs**:
```bash
# System logs
tail -f /exa/logs/logd/cos.log

# Database logs
tail -f /exa/logs/db/Exasol/cored_Exasol_*.log

# View via partition
cosexec 18 -- ls -la /exa/logs/
```

---

### 7. Lock Daemon (lockd) - ID: 19

**Purpose**: Distributed lock management

**Configuration**:
```bash
ID: 19
OWNER: root (0)
GROUP: root (0)
PARENT: 0
FLAGS: RA-I (Restart, Active, Internal)
COMMAND: lockd
```

**Responsibilities**:
- Provides cluster-wide locking
- Coordinates concurrent operations
- Prevents split-brain scenarios
- Ensures consistent state changes

---

### 8. Health Daemon (healthd) - ID: 26

**Purpose**: System health monitoring

**Configuration**:
```bash
ID: 26
OWNER: root (0)
GROUP: root (0)
PARENT: 0
FLAGS: RA-I (Restart, Active, Internal)
COMMAND: healthd
```

**Monitoring**:
- Node health checks
- Resource utilization
- Service availability
- Alerts and notifications

---

### 9. Event Daemon (eventd) - ID: 42

**Purpose**: Event logging and notification

**Configuration**:
```bash
ID: 42
OWNER: root (0)
GROUP: root (0)
PARENT: 0
FLAGS: RA-I (Restart, Active, Internal)
COMMAND: eventd
```

**Features**:
- System event logging
- Alert generation
- Event correlation
- Historical event tracking

---

### 10. Database REST API (db-restapi) - ID: 32

**Purpose**: REST API for database operations

**Configuration**:
```bash
ID: 32
OWNER: root (0)
GROUP: root (0)
PARENT: 0
FLAGS: RANI (Restart, Active, No restart, Internal)
COMMAND: db-restapi
```

**Features**:
- HTTP/REST interface for DB operations
- Alternative to confd_client
- JSON-based responses
- Modern API approach

---

### 11. Cron Daemon (cron) - ID: 17

**Purpose**: Scheduled task execution

**Configuration**:
```bash
ID: 17
OWNER: root (0)
GROUP: root (0)
PARENT: 0
FLAGS: RA-I (Restart, Active, Internal)
COMMAND: cron
```

**Usage**:
- Scheduled maintenance tasks
- Automated backups
- Periodic health checks
- Log rotation

## Checking System Partition Health

### View All System Partitions

```bash
# List all root-level services
cosps -p 0

# With names
cosps -p 0 -m

# Full details
cosps -p 0 -f
```

### Identify Failed Services

```bash
# Check for offline partitions
cosps | awk '$6 != "1/1" {print}'

# Services that should be running
cosps -p 0 | grep -v "1/1"
```

### Monitor Specific Service

```bash
# Watch DWAD
coswatch 27 &

# Watch BucketFS
coswatch 25 &

# Watch ConfD
coswatch 31 &
```

## Restart Behavior

### Auto-Restart Services (R flag)

Most system services auto-restart:
- sshd (16)
- cron (17)
- logd (18)
- lockd (19)
- bucketfsd (25)
- healthd (26)
- dwad (27)
- cos_storage (30)
- confd (31)
- eventd (42)

**Restarting is safe**:
```bash
# Kill service, it will auto-restart
coskill 18  # logd will restart automatically
```

### Manual Restart Required

Some services don't auto-restart or require special handling:
- db-restapi (32) - RANI flag (conflicting flags)

## Service Dependencies

### Dependency Chain

```
1. cos_storage (30) - Must be available first
2. lockd (19) - Distributed locking
3. confd (31) - Configuration management
4. dwad (27) - Database lifecycle (depends on confd)
5. bucketfsd (25) - File system (can start independently)
6. db-restapi (32) - API layer (depends on confd)
```

### Impact of Service Failure

| Service | Impact | Recovery |
|---------|--------|----------|
| **sshd** | No SSH access | Auto-restart |
| **dwad** | All databases stop | Auto-restart, manual DB start |
| **confd** | No config changes | Auto-restart |
| **bucketfsd** | No UDF/driver access | Auto-restart |
| **cos_storage** | Storage operations fail | Auto-restart |
| **logd** | Logging stops | Auto-restart |

## Best Practices


**Monitor critical services**
- DWAD (27) - database lifecycle
- ConfD (31) - configuration
- BucketFS (25) - scripts/drivers


**Don't kill DWAD unless necessary**
- Stops all databases
- Requires manual database restart
- Use `dwad_client` instead


**Use proper tools**
- `confd_client` for database operations
- `dwad_client` for advanced control
- `cosps` for monitoring


**Check logs when issues occur**
- `/exa/logs/cored/` - core service logs
- `/exa/logs/logd/` - system logs
- Service-specific logs

## Related Documentation

- [COS Overview](cos_overview.md)
- [COS Partition Hierarchy](cos_partition_hierarchy.md)
- [COS Database Partitions](cos_database_partitions.md)
- [COS Commands Reference](cos_commands.md)
- [COS Troubleshooting](cos_troubleshooting.md)
- [COS Best Practices](cos_best_practices.md)
