---
tool_name: cos
doc_type: concept
category: COS Architecture
title: "Exasol Directory Structure and File Paths Reference"
summary: "This document provides the **definitive reference** for all Exasol-related directories and file paths. It addresses common path hallucinations by LLMs and provi..."
---
# Exasol Directory Structure and File Paths Reference

**Category:** Administration  
**Topic:** File System, Directory Structure, Logs, Configuration, Troubleshooting  
**Keywords:** paths, directories, logs, configuration, EXAConf, file system, /exa, data, metadata, troubleshooting, file locations  
**Source:** Exasol On-Premise System Analysis

## Overview

This document provides the **definitive reference** for all Exasol-related directories and file paths. It addresses common path hallucinations by LLMs and provides accurate locations for logs, configuration files, data, and other critical system components.

**Critical**: Exasol uses `/exa/` as its primary directory, **NOT** `/var/log/exasol/` or other standard Linux paths.

---

## Table of Contents

1. [Directory Structure Overview](#directory-structure-overview)
2. [/exa Directory Tree](#exa-directory-tree)
3. [Logs (/exa/logs)](#logs-exalogs)
4. [Configuration (/exa/etc)](#configuration-exaetc)
5. [Data (/exa/data)](#data-exadata)
6. [Metadata (/exa/metadata)](#metadata-exametadata)
7. [Spool (/exa/spool)](#spool-exaspool)
8. [System (/exa/sys)](#system-exasys)
9. [Temporary (/exa/tmp)](#temporary-exatmp)
10. [Standard Linux Paths](#standard-linux-paths)
11. [Common Path Mistakes](#common-path-mistakes)
12. [Troubleshooting Path Reference](#troubleshooting-path-reference)

---

## Directory Structure Overview

### Primary Exasol Directory: `/exa/`

**Owner**: `1000:1000` (exadefusr:exausers)  
**Purpose**: Root directory for all Exasol-specific files and data

```
/exa/
├── data/           # Database data, BucketFS, backups
├── etc/            # Configuration files (EXAConf, SSL, etc.)
├── logs/           # ALL Exasol logs (database, system, services)
├── metadata/       # Metadata and storage metadata
├── spool/          # Job queues, coredumps, sync files
├── sys/            # System devices
└── tmp/            # Temporary files and support bundles
```

### Key Characteristics

✅ **Exasol logs**: `/exa/logs/` (NOT `/var/log/exasol/`)  
✅ **Exasol config**: `/exa/etc/` (NOT `/etc/exasol/`)  
✅ **Database data**: `/exa/data/` (NOT `/var/lib/exasol/`)  
✅ **Ownership**: Most Exasol files owned by `exadefusr:exausers`  
✅ **Permissions**: Restrictive (600-700 for sensitive files)

---

## /exa Directory Tree

### Complete Directory Structure

```
/exa/
├── data/
│   ├── bucketfs/
│   │   └── bfsdefault/                # Default BucketFS bucket
│   ├── s3backup/                      # S3 backup data
│   │   ├── .sdfs-upload-state/       # Upload state tracking
│   │   └── Exasol/                    # Database-specific backup data
│   └── storage/                       # EXAStorage data (if applicable)
│
├── etc/
│   ├── EXAConf                        # Primary configuration file
│   ├── EXAConf.0-5                    # Configuration backups (rolling)
│   ├── EXAConf.commited.manually      # Manual commit marker
│   ├── EXAConf.committed.*            # Auto-commit markers
│   ├── dwad/                          # DWAd (Data Warehouse Admin Daemon) config
│   ├── remote_volumes/                # Remote volume configurations
│   ├── ssh/                           # SSH keys and config
│   ├── ssl/                           # SSL certificates and keys
│   ├── bucketfs.cfg_bfsdefault        # BucketFS configuration
│   ├── bucketfs_db.cfg                # BucketFS database mapping
│   ├── cos_storage.conf               # COS storage configuration
│   ├── hostkey                        # Host private key
│   ├── hostkey.pub                    # Host public key
│   ├── known_hosts                    # SSH known hosts
│   ├── license.exasol_license         # License file
│   ├── node_uuid                      # Node UUID
│   └── rsyslog.conf                   # Rsyslog configuration
│
├── logs/
│   ├── cored/                         # Cored daemon logs
│   │   ├── exainit.log               # Exasol initialization log
│   │   ├── exalogrotate.*.log        # Log rotation logs
│   │   └── .logbackup/               # Log backups
│   ├── db/                            # Database logs
│   │   └── Exasol/                    # Database-specific logs
│   │       ├── *_controller_*.0      # Controller logs
│   │       ├── *_PddServer_*.0       # PDD Server logs
│   │       ├── *_ObjectServer_*.0    # Object Server logs
│   │       ├── *_SqlLogServer_*.0    # SQL Log Server logs
│   │       ├── *_ConnectionServer_*.0 # Connection Server logs
│   │       ├── *_LoaderD_*.0         # Loader Daemon logs
│   │       ├── *_EtlProcess_*.0      # ETL Process logs
│   │       └── *_SqlProcess_*.0      # SQL Process logs
│   ├── docker/                        # Docker-related logs (if applicable)
│   ├── logd/                          # Logd daemon logs
│   │   ├── Authentication.log        # Authentication events
│   │   ├── ConfD.log                 # ConfD daemon logs
│   │   ├── DBPool.log                # Database pool logs
│   │   ├── DWAd.log                  # DWAd logs
│   │   ├── EXASolution_*.log         # EXASolution logs
│   │   ├── EXAoperation.log          # EXAoperation logs
│   │   ├── Health.log                # Health check logs
│   │   ├── Lockd.log                 # Lock daemon logs
│   │   ├── Storage.log               # Storage logs
│   │   ├── eventd.log                # Event daemon logs
│   │   ├── exawrap.log               # Exawrap logs
│   │   └── .logbackup/               # Log backups
│   ├── syslog/                        # System logs
│   │   └── .logbackup/               # Syslog backups
│   └── ssl_init.out                   # SSL initialization output
│
├── metadata/
│   ├── dwad/                          # DWAd metadata
│   └── storage/                       # Storage metadata
│       └── .mdbackup/                # Metadata backups
│
├── spool/
│   ├── coredumps/                     # Core dump files
│   ├── jobs/                          # ConfD job management
│   │   ├── archive/                  # Completed job archives
│   │   ├── finish/                   # Finished jobs
│   │   ├── queue/                    # Queued jobs
│   │   ├── run/                      # Running jobs
│   │   └── next.id.*                 # Next job ID tracker
│   └── sync/                          # Configuration sync files
│       ├── EXAConf_*/                # EXAConf sync directories
│
├── sys/
│   └── devices/                       # System device mappings
│
└── tmp/
    └── support/                       # EXAsupport temporary files
```

---

## Logs (/exa/logs)

### Overview

**Path**: `/exa/logs/`  
**Purpose**: Centralized location for ALL Exasol logs

**Critical**: This is the **ONLY** location for Exasol logs. Do **NOT** look in `/var/log/exasol/` (does not exist).

### Log Directory Structure

```
/exa/logs/
├── cored/          # Cored daemon and system initialization
├── db/             # Database process logs
├── docker/         # Docker logs (if using containerized deployment)
├── logd/           # Centralized service logs (ConfD, Authentication, etc.)
├── syslog/         # System-level logs
└── *.out           # One-off output files (e.g., ssl_init.out)
```

### Database Logs (/exa/logs/db/)

**Path**: `/exa/logs/db/<DATABASE_NAME>/`  
**Example**: `/exa/logs/db/Exasol/`

**Log file naming convention**:
```
YYYYMMDD_HHMMSS_<ProcessName>_<ProcessID>.<SequenceNumber>
```

**Database process logs**:

| File Pattern | Description | Typical Size |
|--------------|-------------|--------------|
| `*_controller_*.0` | Main controller process | 3-5 MB |
| `*_PddServer_*.0` | Persistent Data Dictionary server | 1-2 MB |
| `*_ObjectServer_*.0` | Object management server | 1-2 MB |
| `*_SqlLogServer_*.0` | SQL query logging | 20+ MB (largest) |
| `*_ConnectionServer_*.0` | Connection handling | 1-2 MB |
| `*_LoaderD_*.0` | Data loading daemon | 100 KB |
| `*_EtlProcess_*.0` | ETL process logs | Small (KB) |
| `*_SqlProcess_*.0` | SQL execution processes | Small (KB) |

**Example**:
```bash
# View latest controller log
tail -f /exa/logs/db/Exasol/20260120_004038_controller_0.0

# View SQL query log
tail -f /exa/logs/db/Exasol/20260120_004040_SqlLogServer_4.0

# Search for errors in all database logs
grep -i error /exa/logs/db/Exasol/*
```

### Service Logs (/exa/logs/logd/)

**Path**: `/exa/logs/logd/`  
**Purpose**: Centralized logs for Exasol services and daemons

**Key log files**:

| File | Description | Typical Size | Rotation |
|------|-------------|--------------|----------|
| **ConfD.log** | ConfD daemon (cluster management) | 2-3 MB | Daily (10 rotations) |
| **Authentication.log** | Authentication events and user logins | 6-8 MB | Daily (10 rotations) |
| **eventd.log** | Event daemon (alerts, monitoring) | 10-13 MB | Daily (10 rotations) |
| **Health.log** | Health monitoring and checks | 500-600 KB | Daily (10 rotations) |
| **EXASolution_*.log** | EXASolution database logs | Small (KB) | Daily (10 rotations) |
| **Storage.log** | Storage operations | Small (KB) | Daily (10 rotations) |
| **DBPool.log** | Database connection pool | Empty (typically) | Daily (10 rotations) |
| **DWAd.log** | Data Warehouse Admin Daemon | Empty (typically) | Daily (10 rotations) |
| **EXAoperation.log** | EXAoperation web UI | Empty (typically) | Daily (10 rotations) |
| **Lockd.log** | Lock daemon | Empty (typically) | Daily (10 rotations) |
| **exawrap.log** | Exawrap utility logs | Empty (typically) | Daily (10 rotations) |

**Log rotation**:
- Format: `<logfile>.log.<N>` where N = 1-10
- `.log` = current log
- `.log.1` = yesterday
- `.log.10` = 10 days ago

**Examples**:
```bash
# Check ConfD errors
tail -f /exa/logs/logd/ConfD.log | grep -i error

# View authentication attempts
tail -f /exa/logs/logd/Authentication.log

# Check all events from last hour
tail -100 /exa/logs/logd/eventd.log

# View health issues
grep -i "warning\|error\|critical" /exa/logs/logd/Health.log
```

### Cored Logs (/exa/logs/cored/)

**Path**: `/exa/logs/cored/`  
**Purpose**: Core daemon logs and system initialization

**Key files**:
- `exainit.log` - Exasol initialization log (startup, shutdown)
- `exalogrotate.*.log` - Log rotation logs
- `cat.*.log` - Cat process logs
- `.logbackup/` - Archived logs

**Examples**:
```bash
# Check startup issues
tail -100 /exa/logs/cored/exainit.log

# View log rotation errors
grep -i error /exa/logs/cored/exalogrotate*.log
```

### Syslog (/exa/logs/syslog/)

**Path**: `/exa/logs/syslog/`  
**Purpose**: System-level logs forwarded via rsyslog

**Note**: Minimal usage in most Exasol deployments. Application logs use logd instead.

---

## Configuration (/exa/etc)

### Overview

**Path**: `/exa/etc/`  
**Purpose**: All Exasol configuration files  
**Permissions**: 600 (root-only for most files)

### EXAConf - Primary Configuration File

**Path**: `/exa/etc/EXAConf`  
**Permissions**: 600 (root:root)  
**Format**: INI-style configuration

**Critical file**: Contains entire cluster configuration (nodes, databases, storage, networking, etc.)

**Backup versions**:
- `EXAConf.0` - Most recent backup
- `EXAConf.1` - Second most recent
- `EXAConf.2` through `EXAConf.5` - Rolling backups

**Commit markers**:
- `EXAConf.commited.manually` - Manual commit timestamp
- `EXAConf.committed.*` - Auto-commit markers

**View configuration**:
```bash
# View entire config
cat /exa/etc/EXAConf

# View specific section
grep -A 20 "^\[Global\]" /exa/etc/EXAConf

# View database section
grep -A 50 "^\[DB :" /exa/etc/EXAConf

# View node section
grep -A 30 "^\[Node :" /exa/etc/EXAConf
```

**Key sections** in EXAConf:
- `[Global]` - Cluster-wide settings
- `[SSL]` - SSL certificate configuration
- `[Node : <ID>]` - Per-node configuration
- `[DB : <NAME>]` - Database configuration
- `[Volume : <NAME>]` - Storage volume configuration
- `[BucketFS : <NAME>]` - BucketFS configuration

### SSL Certificates

**Path**: `/exa/etc/ssl/`

| File | Description | Permissions |
|------|-------------|-------------|
| `ssl.crt` | SSL certificate | 644 |
| `ssl.key` | SSL private key | 600 |
| `ssl.ca` | Certificate authority (if applicable) | 644 |

**Referenced in EXAConf**:
```ini
[SSL]
    Cert = /exa/etc/ssl/ssl.crt
    CertKey = /exa/etc/ssl/ssl.key
```

### SSH Configuration

**Path**: `/exa/etc/ssh/`

| File | Description |
|------|-------------|
| `config` | SSH client configuration |
| `authorized_keys` | Authorized SSH keys |
| `id_rsa` | Private SSH key |
| `id_rsa.pub` | Public SSH key |

**Also in /exa/etc/**:
- `hostkey` - Host private key (600)
- `hostkey.pub` - Host public key (644)
- `known_hosts` - SSH known hosts

### BucketFS Configuration

**Path**: `/exa/etc/`

| File | Description |
|------|-------------|
| `bucketfs.cfg_bfsdefault` | BucketFS default bucket config |
| `bucketfs_db.cfg` | BucketFS database mapping |

### DWAd Configuration

**Path**: `/exa/etc/dwad/`  
**Purpose**: Data Warehouse Admin Daemon configuration

### Remote Volumes

**Path**: `/exa/etc/remote_volumes/`  
**Purpose**: Remote volume mount configurations

### License

**Path**: `/exa/etc/license.exasol_license`  
**Purpose**: Exasol license file  
**Format**: XML

### Other Configuration Files

| File | Description |
|------|-------------|
| `cos_storage.conf` | COS storage configuration |
| `node_uuid` | Unique node identifier |
| `rsyslog.conf` | Rsyslog configuration |
| `anacron_done` | Anacron completion marker |
| `broadcast_peers.cfg` | Broadcast peer configuration |
| `cored_random` | Cored random seed |
| `init_done` | Initialization completion marker |
| `token.store` | Token storage |

---

## Data (/exa/data)

### Overview

**Path**: `/exa/data/`  
**Purpose**: Database data, BucketFS, backups

```
/exa/data/
├── bucketfs/       # BucketFS data
├── s3backup/       # S3 backup staging
└── storage/        # EXAStorage data (SDFS volumes)
```

### BucketFS (/exa/data/bucketfs)

**Path**: `/exa/data/bucketfs/<BUCKET_NAME>/`  
**Example**: `/exa/data/bucketfs/bfsdefault/`

**Purpose**: Object storage for:
- UDF scripts (Python, R, Java, Lua)
- JDBC/ODBC drivers
- Script language containers
- Custom libraries and files

**Access**:
- HTTP/HTTPS: Via bucket HTTP service
- File system: Direct access on nodes

**Example**:
```bash
# List BucketFS contents
ls -lh /exa/data/bucketfs/bfsdefault/

# Check bucket size
du -sh /exa/data/bucketfs/bfsdefault/
```

### S3 Backup (/exa/data/s3backup)

**Path**: `/exa/data/s3backup/`  
**Purpose**: Staging area for S3 backups

**Structure**:
```
/exa/data/s3backup/
├── .sdfs-upload-state/     # Upload state tracking
└── <DATABASE_NAME>/         # Per-database backup data
```

**Example**:
```bash
# Check backup staging size
du -sh /exa/data/s3backup/

# View upload state
ls -lh /exa/data/s3backup/.sdfs-upload-state/
```

### Storage (/exa/data/storage)

**Path**: `/exa/data/storage/`  
**Purpose**: EXAStorage SDFS volume data (if using on-premise storage)

**Note**: This may be empty in cloud deployments using instance store or EBS.

---

## Metadata (/exa/metadata)

### Overview

**Path**: `/exa/metadata/`  
**Purpose**: System and storage metadata

```
/exa/metadata/
├── dwad/           # DWAd metadata
└── storage/        # Storage metadata
    └── .mdbackup/  # Metadata backups
```

### Storage Metadata

**Path**: `/exa/metadata/storage/`  
**Purpose**: SDFS storage metadata, volume information

**Backup**: `/exa/metadata/storage/.mdbackup/`

---

## Spool (/exa/spool)

### Overview

**Path**: `/exa/spool/`  
**Purpose**: Job queues, coredumps, synchronization files

```
/exa/spool/
├── coredumps/      # Core dump files from crashes
├── jobs/           # ConfD job management
│   ├── archive/    # Completed jobs (large directory)
│   ├── finish/     # Finished jobs
│   ├── queue/      # Queued jobs waiting to run
│   ├── run/        # Currently running jobs
│   └── next.id.*   # Next job ID tracker
└── sync/           # Configuration synchronization
    └── EXAConf_*/  # EXAConf sync directories
```

### Job Management (/exa/spool/jobs)

**Path**: `/exa/spool/jobs/`  
**Purpose**: ConfD job lifecycle management

**Workflow**:
1. **queue/** - Job created, waiting to run
2. **run/** - Job currently executing
3. **finish/** - Job completed, pending archival
4. **archive/** - Job archived (can be very large)

**Examples**:
```bash
# Check running jobs
ls -lh /exa/spool/jobs/run/

# Check queued jobs
ls -lh /exa/spool/jobs/queue/

# Count archived jobs
find /exa/spool/jobs/archive/ -type f | wc -l

# View recent job
ls -lth /exa/spool/jobs/archive/ | head -5
```

### Coredumps (/exa/spool/coredumps)

**Path**: `/exa/spool/coredumps/`  
**Purpose**: Core dump files from process crashes

**Usage**: Debugging serious process failures

### Sync (/exa/spool/sync)

**Path**: `/exa/spool/sync/`  
**Purpose**: Configuration synchronization between nodes

**Contains**: EXAConf synchronization directories

---

## System (/exa/sys)

**Path**: `/exa/sys/`  
**Purpose**: System device mappings

```
/exa/sys/
└── devices/        # Device information
```

**Usage**: Internal system references, typically not accessed directly.

---

## Temporary (/exa/tmp)

**Path**: `/exa/tmp/`  
**Purpose**: Temporary files and support bundles

```
/exa/tmp/
└── support/        # EXAsupport temporary files
```

### EXAsupport Files

**Path**: `/exa/tmp/support/`  
**Purpose**: Temporary files generated by EXAsupport tool

**Contents**: Diagnostic bundles, system info, logs collected for support cases

**Example**:
```bash
# List support bundles
ls -lh /exa/tmp/support/
```

---

## Standard Linux Paths

### System Logs (/var/log)

**Path**: `/var/log/`  
**Purpose**: Standard Linux system logs (NOT Exasol logs)

**Important**: Exasol does **NOT** use `/var/log/` for application logs.

**Contents** (non-Exasol):
- `alternatives.log` - Debian alternatives
- `apt/` - APT package manager logs
- `bootstrap.log` - System bootstrap
- `dpkg.log` - Package installation
- `btmp` - Failed login attempts
- `wtmp` - Login records
- `lastlog` - Last login per user
- `faillog` - Failed login log

**No Exasol logs here!**

### Home Directories (/home)

**Exasol user**: `/home/exadefusr/` (may vary)  
**Admin user**: `/home/<admin_user>/`

### Root Directory (/)

**Exasol-specific files** in root:
- `/.license.xml` - License file copy
- `/check_sqlquery` - SQL query checker binary

---

## Common Path Mistakes

### ❌ Incorrect Paths (DO NOT USE)

| Incorrect Path | Correct Path | Notes |
|----------------|--------------|-------|
| `/var/log/exasol/` | `/exa/logs/` | Exasol does NOT use /var/log |
| `/etc/exasol/` | `/exa/etc/` | Config is in /exa, not /etc |
| `/var/lib/exasol/` | `/exa/data/` | Data is in /exa, not /var/lib |
| `/var/spool/exasol/` | `/exa/spool/` | Spool is in /exa, not /var |
| `/opt/exasol/` | `/exa/` | No /opt directory used |
| `/usr/local/exasol/` | `/exa/` | Not in /usr/local |

### Common Hallucinations

**LLMs often incorrectly suggest**:
- Backup logs in `/var/log/exasol/backup_*.log` ❌
  - **Correct**: Check ConfD logs in `/exa/logs/logd/ConfD.log`
  
- Database logs in `/var/log/exasol/database.log` ❌
  - **Correct**: `/exa/logs/db/<DATABASE_NAME>/*`
  
- Configuration in `/etc/exasol/exasol.conf` ❌
  - **Correct**: `/exa/etc/EXAConf`
  
- License in `/opt/exasol/license.xml` ❌
  - **Correct**: `/exa/etc/license.exasol_license`

---

## Troubleshooting Path Reference

### Common Troubleshooting Scenarios

#### Backup Issues

**Symptom**: Backup stuck or failing

**Check these paths**:
```bash
# ConfD logs for backup status
tail -f /exa/logs/logd/ConfD.log | grep -i backup

# Check backup job status
ls -lh /exa/spool/jobs/run/ | grep backup

# View archived backup jobs
ls -lth /exa/spool/jobs/archive/ | grep backup | head -10

# Check S3 backup staging
du -sh /exa/data/s3backup/
ls -lh /exa/data/s3backup/.sdfs-upload-state/

# Check storage logs
tail -f /exa/logs/logd/Storage.log
```

#### Database Startup Issues

**Symptom**: Database won't start

**Check these paths**:
```bash
# Controller log
tail -100 /exa/logs/db/Exasol/*controller*

# ConfD log
tail -100 /exa/logs/logd/ConfD.log | grep -i error

# Initialization log
tail -100 /exa/logs/cored/exainit.log

# Check EXAConf
grep -A 50 "^\[DB :" /exa/etc/EXAConf
```

#### Authentication Issues

**Symptom**: Login failures

**Check these paths**:
```bash
# Authentication log
tail -f /exa/logs/logd/Authentication.log

# Connection server log
tail -f /exa/logs/db/Exasol/*ConnectionServer*
```

#### Performance Issues

**Symptom**: Slow queries

**Check these paths**:
```bash
# SQL query log (largest file, lots of detail)
tail -f /exa/logs/db/Exasol/*SqlLogServer*

# SQL process logs
ls -lh /exa/logs/db/Exasol/*SqlProcess*
```

#### BucketFS Issues

**Symptom**: Can't access UDF scripts or drivers

**Check these paths**:
```bash
# BucketFS data
ls -lh /exa/data/bucketfs/bfsdefault/

# BucketFS config
cat /exa/etc/bucketfs.cfg_bfsdefault
cat /exa/etc/bucketfs_db.cfg
```

#### SSL Certificate Issues

**Symptom**: SSL connection failures

**Check these paths**:
```bash
# SSL cert and key
ls -lh /exa/etc/ssl/

# SSL initialization output
cat /exa/logs/ssl_init.out

# EXAConf SSL section
grep -A 10 "^\[SSL\]" /exa/etc/EXAConf
```

#### Cluster Configuration Issues

**Symptom**: Node not joining cluster, configuration errors

**Check these paths**:
```bash
# EXAConf
cat /exa/etc/EXAConf

# EXAConf backups (if need to revert)
ls -lh /exa/etc/EXAConf.*

# Sync status
ls -lh /exa/spool/sync/

# ConfD log
tail -100 /exa/logs/logd/ConfD.log
```

#### Health Monitoring

**Symptom**: Cluster health warnings

**Check these paths**:
```bash
# Health log
tail -f /exa/logs/logd/Health.log

# Event log
tail -f /exa/logs/logd/eventd.log | grep -i "warning\|error\|critical"
```

#### Storage Issues

**Symptom**: Disk full, storage errors

**Check these paths**:
```bash
# Storage log
tail -f /exa/logs/logd/Storage.log

# Storage metadata
ls -lh /exa/metadata/storage/

# Check actual disk usage
df -h
du -sh /exa/data/
du -sh /exa/logs/
du -sh /exa/spool/jobs/archive/
```

#### ConfD Job Issues

**Symptom**: Job stuck, not running

**Check these paths**:
```bash
# Running jobs
ls -lh /exa/spool/jobs/run/

# Queued jobs
ls -lh /exa/spool/jobs/queue/

# ConfD log
tail -f /exa/logs/logd/ConfD.log

# Recent finished jobs
ls -lth /exa/spool/jobs/finish/ | head -10
```

---

## Quick Reference

### Essential Paths

```bash
# Configuration
/exa/etc/EXAConf                        # Primary config file

# Logs
/exa/logs/logd/ConfD.log               # ConfD daemon
/exa/logs/logd/Authentication.log      # Auth events
/exa/logs/db/<DB_NAME>/*               # Database logs
/exa/logs/cored/exainit.log            # Initialization

# Data
/exa/data/bucketfs/bfsdefault/         # BucketFS data
/exa/data/s3backup/                    # S3 backup staging

# Jobs
/exa/spool/jobs/run/                   # Running jobs
/exa/spool/jobs/queue/                 # Queued jobs
/exa/spool/jobs/archive/               # Completed jobs

# SSL
/exa/etc/ssl/ssl.crt                   # SSL certificate
/exa/etc/ssl/ssl.key                   # SSL private key
```

### Disk Space Checklist

```bash
# Check /exa disk usage
du -sh /exa/*

# Check large subdirectories
du -sh /exa/logs/*
du -sh /exa/data/*
du -sh /exa/spool/jobs/*

# Find largest files
find /exa -type f -size +100M -exec ls -lh {} \; | sort -k5 -rh | head -20

# Clean up old logs
find /exa/logs/cored -name "*.log" -mtime +30 -delete

# Clean up old job archives (CAUTION!)
find /exa/spool/jobs/archive -type f -mtime +90 -delete
```

### Log Rotation

**Automatic rotation**: Logs in `/exa/logs/logd/` rotate daily, keeping 10 versions.

**Manual cleanup**:
```bash
# Remove old log backups in cored
find /exa/logs/cored/.logbackup -mtime +30 -delete

# Remove old database logs
find /exa/logs/db -name "*.0" -mtime +30 -delete
```

---

## Related Documentation

- [ConfD](https://docs.exasol.com/db/latest/confd/confd.htm)
- [EXAConf Configuration](https://docs.exasol.com/db/latest/confd/exaconf.htm)
- [BucketFS](https://docs.exasol.com/db/latest/database_concepts/bucketfs/bucketfs.htm)
- [Backup and Restore](https://docs.exasol.com/db/latest/administration/on-premise/backup_restore.htm)
- [System Tables](https://docs.exasol.com/db/latest/sql_references/system_tables/statistical_system_tables.htm)

## Common Questions

- Where are Exasol logs located?
- Where is the Exasol configuration file?
- Where are database logs stored?
- Where are backup logs in Exasol?
- How do I find ConfD logs?
- Where is EXAConf located?
- Where are BucketFS files stored?
- Where are SSL certificates in Exasol?
- Where are job logs in Exasol?
- How do I check disk space for Exasol?
- Where are authentication logs in Exasol?
- Where are database process logs?
- Where is the license file in Exasol?
- Where are coredumps stored in Exasol?
- Where are temporary files in Exasol?
- What is the /exa directory structure?
- Why can't I find logs in /var/log/exasol/?
- Where are Exasol data files stored?
- Where is the SSH configuration for Exasol?
- Where are metadata files in Exasol?

## Summary

**Key Takeaways:**

✅ **ALL Exasol logs**: `/exa/logs/` (NOT `/var/log/exasol/`)  
✅ **Primary config**: `/exa/etc/EXAConf` (NOT `/etc/exasol/`)  
✅ **Database data**: `/exa/data/` (NOT `/var/lib/exasol/`)  
✅ **Job management**: `/exa/spool/jobs/` (NOT `/var/spool/exasol/`)

**Directory Purposes:**
- `/exa/logs/` - All logs (db, services, system)
- `/exa/etc/` - All configuration
- `/exa/data/` - Data, BucketFS, backups
- `/exa/metadata/` - System metadata
- `/exa/spool/` - Jobs, coredumps, sync
- `/exa/sys/` - System devices
- `/exa/tmp/` - Temporary files

**Common Mistakes:**
- Looking for logs in `/var/log/exasol/` ❌
- Looking for config in `/etc/exasol/` ❌
- Looking for data in `/var/lib/exasol/` ❌

**Always remember**: Exasol uses `/exa/` for everything!
