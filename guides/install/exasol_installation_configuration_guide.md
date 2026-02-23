# Exasol Installation and Configuration Guide

## Overview

This guide provides comprehensive step-by-step instructions for installing and configuring an Exasol cluster. It covers everything from initial SSH access through complete database setup, including storage configuration, licensing, backup setup, and BucketFS configuration.

**Target Audience**: System administrators, DevOps engineers, database administrators

**Prerequisites**:
- SSH access to Exasol data nodes
- Private SSH key file
- Exasol installation binaries (e.g., `exasol-8.32.0.tar.gz`)
- c4 deployment tool
- Valid Exasol license file
- BucketFS archive (optional, for JDBC drivers and virtual schemas)

**Related Documentation**:
- [c4 Comprehensive Guide](c4_comprehensive_guide.md)
- [Exasol Directory Structure](exasol_directory_structure.md)
- [Exasol Node Management](exasol_node_management.md)
- [Exasol Storage Management](exasol_storage_management.md)
- [Exasol Update Procedure](exasol_update_procedure.md)

---

## Table of Contents

1. [Initial Access and Connection](#1-initial-access-and-connection)
2. [Storage Preparation](#2-storage-preparation)
3. [File Transfer](#3-file-transfer)
4. [Pre-Installation Configuration](#4-pre-installation-configuration)
5. [Exasol Installation](#5-exasol-installation)
6. [System Validation](#6-system-validation)
7. [License Management](#7-license-management)
8. [Database Configuration](#8-database-configuration)
9. [Backup Configuration](#9-backup-configuration)
10. [BucketFS Setup](#10-bucketfs-setup)
11. [Troubleshooting](#11-troubleshooting)

---

## 1. Initial Access and Connection

### 1.1 SSH Connection to Data Node

**Purpose**: Establish secure SSH access to Exasol data nodes for installation and configuration.

#### Connection Command

```bash
ssh -i "path/to/your/key.txt" adminuser@10.248.15.136
```

**Parameters**:
- `-i "path/to/your/key.txt"`: Path to your SSH private key
- `adminuser`: Default username for data node access
- `10.248.15.136`: IP address of the target data node (environment-specific)

#### Platform-Specific Examples

**Windows PowerShell**:
```powershell
ssh -i "C:\Users\Dren.Daka\Desktop\key.txt" adminuser@10.248.15.136
```

**macOS/Linux**:
```bash
ssh -i ~/keys/exasol_key.txt adminuser@10.248.15.136
```

#### Key File Permissions

**Critical**: SSH private keys must have restrictive permissions.

```bash
# Linux/macOS - Set correct permissions
chmod 600 key.txt

# Verify permissions
ls -l key.txt
# Expected: -rw------- (600)
```

**Common Issues**:
- **Error**: "Permissions 0644 for 'key.txt' are too open"
  - **Solution**: Run `chmod 600 key.txt`
- **Error**: "Permission denied (publickey)"
  - **Solution**: Verify you're using the correct key and username

---

## 2. Storage Preparation

### 2.1 LVM Disk Configuration

**Purpose**: Create Logical Volume Manager (LVM) disks for Exasol storage volumes.

**⚠️ WARNING**: LVM operations can result in data loss. Always have backups before proceeding.

#### Create Physical Volumes

```bash
# Create physical volumes on raw devices
sudo pvcreate /dev/sdc /dev/sdd

# Verify physical volumes
sudo pvdisplay
```

#### Create Volume Groups

```bash
# Create volume groups for Exasol disks
sudo vgcreate exa1 /dev/sdc
sudo vgcreate exa2 /dev/sdd

# Verify volume groups
sudo vgdisplay
```

#### Create Logical Volumes

```bash
# Allocate 100% of free space in volume groups
sudo lvcreate -l 100%FREE exa1
sudo lvcreate -l 100%FREE exa2

# Verify logical volumes
sudo lvdisplay
```

**Expected Result**: Logical volumes `/dev/exa1/lvol0` and `/dev/exa2/lvol0` created.

#### Best Practices

- **Disk Selection**: Use dedicated physical disks for Exasol data volumes
- **RAID Configuration**: Consider RAID 10 for production environments
- **Disk Performance**: Use SSDs or high-performance HDDs for better performance
- **Capacity Planning**: Allocate 2-3x the expected data size for growth

**Related**: See [Exasol Storage Management](exasol_storage_management.md) for advanced storage operations.

---

## 3. File Transfer

### 3.1 Transfer Installation Files

**Purpose**: Copy Exasol binaries, c4 tool, license, and BucketFS archives to the first data node.

#### Required Files

- `exasol-8.32.0.tar.gz` - Exasol installation package
- `c4` - Cluster deployment tool
- `license-qa` - Exasol license file
- `fresenius-bucketfs.tar` - BucketFS archive (optional)

#### Transfer Commands

```bash
# Transfer BucketFS archive
scp -O -i "C:\Users\Dren.Daka\Desktop\qakey.txt" \
  C:\Users\Dren.Daka\Desktop\fresenius-bucketfs.tar \
  adminuser@10.248.28.132:~/

# Transfer Exasol installation package
scp -O -i "C:\Users\Dren.Daka\Desktop\qakey.txt" \
  "C:\Users\Dren.Daka\Downloads\exasol-8.32.0.tar.gz" \
  adminuser@10.248.28.132:~/

# Transfer c4 tool
scp -O -i "C:\Users\Dren.Daka\Desktop\qakey.txt" \
  "C:\Users\Dren.Daka\Downloads\c4" \
  adminuser@10.248.28.132:~/

# Transfer license file
scp -O -i "C:\Users\Dren.Daka\Desktop\qakey.txt" \
  "C:\Users\Dren.Daka\Downloads\license-qa" \
  adminuser@10.248.28.132:~/
```

**Parameter Explanation**:
- `-O`: Use legacy SCP protocol (required for some configurations)
- `-i`: SSH private key path
- Source path: Local file location
- Destination: `user@host:~/` (home directory on remote host)

#### Verification

```bash
# SSH to the data node
ssh -i "C:\Users\Dren.Daka\Desktop\qakey.txt" adminuser@10.248.28.132

# Verify transferred files
ls -lh ~/
```

**Expected Output**:
```
-rw-r--r-- 1 adminuser adminuser  1.2G Jan 23 10:30 exasol-8.32.0.tar.gz
-rwxr-xr-x 1 adminuser adminuser   45M Jan 23 10:31 c4
-rw-r--r-- 1 adminuser adminuser  2.1K Jan 23 10:31 license-qa
-rw-r--r-- 1 adminuser adminuser  850M Jan 23 10:29 fresenius-bucketfs.tar
```

---

## 4. Pre-Installation Configuration

### 4.1 SSH Key Pair Setup (Optional)

**Purpose**: Create SSH keys for passwordless access between Exasol nodes.

```bash
# Generate ECDSA key pair
ssh-keygen -t ecdsa

# Press Enter to accept default location (~/.ssh/id_ecdsa)
# Press Enter twice for no passphrase
```

#### Distribute Public Key

```bash
# Add public key to authorized_keys (on all data nodes)
cat ~/.ssh/id_ecdsa.pub >> ~/.ssh/authorized_keys

# Set correct permissions
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh
```

### 4.2 Firewall Configuration

**⚠️ WARNING**: Disabling firewall should only be done in controlled network environments.

```bash
# Stop firewall (on all nodes)
sudo systemctl stop ufw

# Disable firewall at boot (on all nodes)
sudo systemctl disable ufw

# Verify firewall status
sudo systemctl status ufw
```

**Production Recommendation**: Instead of disabling firewall, configure appropriate rules:
- TCP 8563 (database connection)
- TCP 20002 (COS SSH access)
- TCP 2581-2582 (BucketFS HTTPS)
- TCP 443 (HTTPS)

### 4.3 c4 Configuration File

**Purpose**: Define cluster deployment parameters in a c4 configuration file.

Create file `c4-config`:

```bash
# Node IP addresses (space-separated)
CCC_HOST_ADDRS="10.248.28.132 10.248.28.133 10.248.28.134"

# Data disk devices (comma-separated, per node)
CCC_HOST_DATADISK=/dev/exa1/lvol0,/dev/exa2/lvol0

# SSH connection settings
CCC_HOST_IMAGE_USER=adminuser
CCC_HOST_IMAGE_PASSWORD=
CCC_HOST_KEY_PAIR_FILE=id_ecdsa

# Exasol version to install
CCC_PLAY_WORKING_COPY=@exasol-8.32.0

# Database password for default 'sys' user
CCC_PLAY_DB_PASSWORD=exasol456

# Number of reserve nodes (standby nodes)
CCC_PLAY_RESERVE_NODES=1
```

**Configuration Parameters**:

| Parameter | Description | Example |
|-----------|-------------|---------|
| `CCC_HOST_ADDRS` | IP addresses of all cluster nodes | `"10.248.28.132 10.248.28.133 10.248.28.134"` |
| `CCC_HOST_DATADISK` | LVM logical volumes for data storage | `/dev/exa1/lvol0,/dev/exa2/lvol0` |
| `CCC_HOST_IMAGE_USER` | SSH username for node access | `adminuser` |
| `CCC_HOST_KEY_PAIR_FILE` | SSH key filename (in ~/.ssh/) | `id_ecdsa` |
| `CCC_PLAY_WORKING_COPY` | Exasol version reference | `@exasol-8.32.0` |
| `CCC_PLAY_DB_PASSWORD` | Default 'sys' user password | `exasol456` |
| `CCC_PLAY_RESERVE_NODES` | Number of standby nodes | `1` |

**Best Practices**:
- Use strong passwords for `CCC_PLAY_DB_PASSWORD`
- Keep 1 reserve node for every 2-3 active nodes
- Verify IP addresses are accessible before installation

**Related**: See [c4 Comprehensive Guide](c4_comprehensive_guide.md) for all configuration options.

---

## 5. Exasol Installation

### 5.1 Run c4 Installation

**Purpose**: Deploy Exasol cluster using c4 tool.

```bash
# Make c4 executable
chmod +x c4

# Run installation
./c4 host play -i c4-config
```

### 5.2 Installation Process

The installation proceeds through several phases:

#### Phase 1: Pre-Installation Checks
```
INFO[2025-01-23 16:12:37] Creating new host deployment...
INFO[2025-01-23 16:12:37] Done reading configuration.
```

**What happens**:
- Configuration validation
- IP address verification
- SSH connectivity checks

#### Phase 2: OS Preparation
```
INFO[2025-01-23 16:12:39] Initial OS preparation
INFO[2025-01-23 16:12:41] Veryfing OS configuration...
INFO[2025-01-23 16:12:41] OS on all hosts is configured properly
```

**What happens**:
- Root access verification
- Disk availability checks
- OS compatibility validation

#### Phase 3: Package Distribution
```
INFO[2025-01-23 16:12:42] Fetching Exasol packages...
INFO[2025-01-23 16:12:42] Copying Exasol packages to remote hosts...
INFO[2025-01-23 16:13:44] Done copying Exasol packages to remote hosts.
```

**What happens**:
- Exasol packages transferred to all nodes
- Typically takes 1-2 minutes per node

#### Phase 4: Package Extraction
```
INFO[2025-01-23 16:13:44] Extracting packages...
INFO[2025-01-23 16:16:41] Done extracting packages.
```

**What happens**:
- Extraction of Exasol binaries
- Takes 2-3 minutes depending on hardware

#### Phase 5: Cluster Initialization
```
INFO[2025-01-23 16:16:41] Installing the secret and SSH keys...
INFO[2025-01-23 16:16:43] Running installation...
```

**What happens**:
- SSH keys distributed
- Cluster secrets established
- Node communication setup

#### Phase 6: Final Stage
```
INFO[2025-01-23 16:16:46] Done running installation.
|||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
  The final steps of the Exasol installation procedure were successfully
  started on remote hosts now.
  It will take yet some time to complete (several minutes).
  The installation is finished when every node reaches stage 'd' (see 'c4 ps').
```

**What happens**:
- Background installation continues on nodes
- Nodes progress through stages: a → b → c → d
- Stage 'd' indicates installation complete

### 5.3 Monitor Installation Progress

```bash
# Check node stages
./c4 ps

# Watch journalctl logs on a node (via SSH)
sudo journalctl -f
```

**Expected Output** (installation complete):
```
      N  PLAY_ID  NODE  MEDIUM  INSTANCE  DB_VERSION  EXTERNAL_IP    INTERNAL_IP    STAGE  STATE  UPTIME    TTL
  ┌─  1  -        11    host    -         -           10.248.28.132  10.248.28.132  d      -      00:05:23  +∞
  │   1  -        12    host    -         -           10.248.28.133  10.248.28.133  d      -      00:05:23  +∞
  └─  1  -        13    host    -         -           10.248.28.134  10.248.28.134  d      -      00:05:23  +∞
```

**Stage Meanings**:
- **a**: Initial stage
- **b**: OS setup complete
- **c**: Exasol services starting
- **d**: Installation complete, cluster ready

### 5.4 Connect to COS

**Purpose**: Access Cluster Operating System (COS) for configuration.

```bash
# Connect to COS on any node
ssh -p 20002 root@10.248.28.132

# Password: CCC_PLAY_DB_PASSWORD from c4-config (e.g., exasol456)
```

**Alternative using c4**:
```bash
./c4 connect -t 1/cos
```

---

## 6. System Validation

### 6.1 Check Service Status

**From Host OS** (before SSH to COS):

```bash
# Check c4 service status
systemctl status c4_cloud_command | head -10
```

**Expected Output**:
```
● c4_cloud_command.service - Exasol Cloud Command
     Loaded: loaded (/etc/systemd/system/c4_cloud_command.service; enabled)
     Active: active (running) since Thu 2025-01-23 16:16:50 UTC; 10min ago
```

### 6.2 Validate Node Status

**From COS** (after SSH to COS via `ssh -p 20002 root@NODE_IP`):

```bash
# Check all nodes and processes
cosps -N
```

**Expected Output** (healthy cluster):
```
ID      ROOT NODE       STATE
11:     n11     online
12:     n12     online
13:     n13     online

ID      OWNER   GROUP   PARENT  FLAGS   NODES           OFFLINE         COMMAND
18          0       0        0   R--I   11,12,13        -               sshd
19          0       0        0   RA-I   11,12,13        -               cron
20          0       0        0   RA-I   11,12,13        -               logd
21          0       0        0   RA-I   11,12,13        -               lockd
29        500     500        0   RAEI   11,12,13        -               bucketfsd-bfsdefault
30          0       0        0   RA-I   11,12,13        -               healthd
31          0       0        0   RA-I   11,12,13        -               dwad
34          0       0        0   RA-I   11,12,13        -               cos_storage
35          0       0        0   RA-I   11,12,13        -               confd
36          0       0        0   RANI   11,12,13        -               db-restapi
39        500     500       31   --EK   11,12           -               controller-Exasol
40        500     500       39   --NI   11,12           -               pddserver-Exasol
41        500     500       39   --NI   11,12           -               objectserver-Exasol
43        500     500       39   --NI   11,12           -               exasqllog-Exasol
44        500     500       39   --NI   11,12           -               loaderd-Exasol
45        500     500       39   --NI   11,12           -               exacs-Exasol
46        500     500       39   --NI   11,12           -               exaetl-1024-Exasol
47        500     500       39   --NI   11,12           -               exasql-7-Exasol
49          0       0        0   RA-I   11,12,13        -               eventd
```

**Key Services**:
- **sshd**: SSH daemon
- **confd**: Configuration daemon (manages cluster config)
- **dwad**: Database and warehouse daemon
- **logd**: Logging daemon
- **bucketfsd**: BucketFS daemon
- **controller-{DB}**: Database controller process
- **exasql-{N}-{DB}**: SQL query processes

**Process States** (FLAGS):
- **R**: Running
- **A**: Auto-restart enabled
- **E**: Enhanced mode
- **I**: Important process
- **K**: Kill signal sent
- **N**: Normal mode

### 6.3 Validate Disk Health

```bash
# Check storage disk health
csinfo -H
```

**Expected Output** (healthy disks):
```
=== Node 11 (n11) ===
    HDD 0 (/dev/exa1/lvol0, disk1) : ONLINE
    HDD 1 (/dev/exa2/lvol0, disk1) : ONLINE

=== Node 12 (n12) ===
    HDD 0 (/dev/exa1/lvol0, disk1) : ONLINE
    HDD 1 (/dev/exa2/lvol0, disk1) : ONLINE

=== Node 13 (n13) ===
    HDD 0 (/dev/exa1/lvol0, disk1) : ONLINE
    HDD 1 (/dev/exa2/lvol0, disk1) : ONLINE
```

**Disk States**:
- **ONLINE**: Disk healthy and operational
- **OFFLINE**: Disk unavailable (critical issue)
- **IO_ERRORS**: Disk experiencing errors (warning)

**Related**: See [Exasol Storage Management](exasol_storage_management.md) for disk troubleshooting.

---

## 7. License Management

### 7.1 Upload License

**Purpose**: Apply Exasol license to enable database creation and remove trial limitations.

**From Host OS** (using c4):

```bash
# Upload license via c4 and confd_client
cat license-qa | c4 connect -t1/cos -- confd_client license_upload license: '\""{< -}\""'
```

**Parameter Explanation**:
- `cat license-qa`: Read license file content
- `c4 connect -t1/cos --`: Connect to COS and execute command
- `confd_client license_upload`: ConfD command to upload license
- `license: '\""{< -}\""'`: Read from stdin (pipe)

**Alternative (from COS directly)**:

```bash
# SSH to COS
ssh -p 20002 root@NODE_IP

# Upload license
confd_client license_upload license: "$(cat ~/license-qa)"
```

### 7.2 Verify License

```bash
# Check license information
confd_client license_info
```

**Expected Output**:
```yaml
Contract:
  comment: Unlimited license with an expiration date.
  company_name: Exasol
  distributor: Exasol
  distributor_id: 1
  expiration_date: '2024-08-02'
  license_id: 2
Exasol_DB_license:
  schema_version: 1
Limits:
  max_db_mem_size_in_gb: Unlimited
  max_db_raw_data_size_in_gb: Unlimited
  max_nodes_per_cluster: Unlimited
  max_num_clusters: Unlimited
```

**License Types**:
- **Unlimited**: No size/node restrictions (typically production)
- **Trial**: Time-limited with size restrictions
- **Developer**: Limited nodes and data size

---

## 8. Database Configuration

### 8.1 Remove Default Database (Optional)

**Purpose**: Remove default "Exasol" database and "DataVolume1" if not needed.

```bash
# Stop default database
confd_client db_stop db_name: Exasol

# Delete default database
confd_client db_delete db_name: Exasol

# Delete default volume
confd_client st_volume_delete vname: DataVolume1
```

**Expected Output**:
```
OK
OK
OK
```

### 8.2 Create Data Volume

**Purpose**: Create storage volume for database data.

```bash
# Create data volume
confd_client st_volume_create \
  name: DataVolume1 \
  disk: disk1 \
  type: data \
  size: '600 GiB' \
  nodes: '[11,12]' \
  redundancy: 2 \
  owner: '[500,500]'
```

**Parameters**:
- `name`: Volume name (alphanumeric, no spaces)
- `disk`: Disk identifier (`disk1`, `disk2`, etc.)
- `type`: Volume type (`data`, `archive`)
- `size`: Volume size with unit (`GiB`, `TiB`)
- `nodes`: Node IDs where volume resides
- `redundancy`: Mirror count (1=no redundancy, 2=mirrored)
- `owner`: UID and GID (`[500,500]` = exasol user)

**Expected Output**:
```
vid: 0
```

**Best Practices**:
- **Redundancy**: Use `redundancy: 2` for production (data mirroring)
- **Size**: Leave 20% free space for operations and growth
- **Nodes**: Distribute across multiple nodes for fault tolerance

**Related**: See [Exasol Storage Management](exasol_storage_management.md) for volume management.

### 8.3 Create Database

**Purpose**: Create Exasol database with custom configuration.

#### Basic Database

```bash
confd_client db_create \
  db_name: mydb \
  version: 8.32.0 \
  data_volume_name: DataVolume1 \
  mem_size: '23 GiB' \
  port: 8563 \
  nodes: '[11, 12, 13]' \
  num_active_nodes: 2 \
  owner: '[500, 500]' \
  enable_auditing: true
```

#### Database with LDAP Authentication

```bash
confd_client db_create \
  db_name: qa \
  version: 8.32.0 \
  data_volume_name: DataVolume1 \
  mem_size: '23 GiB' \
  port: 8563 \
  nodes: '[11, 12, 13]' \
  num_active_nodes: 2 \
  owner: '[500, 500]' \
  enable_auditing: true \
  ldap_server: 'ldaps://ads.example.com:3269,ldaps://ads.example.com:636'
```

**Parameters**:

| Parameter | Description | Example |
|-----------|-------------|---------|
| `db_name` | Database name | `qa` |
| `version` | Exasol version | `8.32.0` |
| `data_volume_name` | Data volume to use | `DataVolume1` |
| `mem_size` | RAM allocated to database | `23 GiB` |
| `port` | Database connection port | `8563` |
| `nodes` | Nodes available for database | `[11, 12, 13]` |
| `num_active_nodes` | Number of active nodes | `2` |
| `owner` | UID and GID | `[500, 500]` |
| `enable_auditing` | Enable audit logging | `true` |
| `ldap_server` | LDAP server URLs (comma-separated) | `ldaps://...` |

**LDAP Server Format**:
- `ldap://hostname:port` - Unencrypted LDAP
- `ldaps://hostname:port` - LDAP over SSL/TLS (recommended)
- Multiple servers: Comma-separated for failover

**Expected Output**:
```
OK
```

**Memory Sizing**:
- **Minimum**: 4 GiB per database
- **Recommended**: 50-75% of total node RAM
- **Max**: Leave 4-8 GiB for OS operations

**Related**: See [Exasol Sizing Guidelines](exasol_sizing_guidelines.md) for capacity planning.

### 8.4 Start Database

```bash
# Start database
confd_client db_start db_name: qa
```

**Expected Output**:
```
OK
WARNING: DB memory 23552 MiB is reduced to allowed maximum of 23324 MiB.
```

**Note**: Memory warning is normal - ConfD automatically adjusts to available RAM.

### 8.5 Verify Database Status

```bash
# Check database state
confd_client db_info db_name: qa | grep -i state
```

**Expected Output**:
```
state: running
```

**Database States**:
- **running**: Database operational
- **stopped**: Database stopped (normal)
- **starting**: Database starting (transient)
- **stopping**: Database stopping (transient)
- **failed**: Database in error state (investigate logs)

**Troubleshooting Failed State**:
1. Check logs: `/exa/logs/cored/DataStorage.log`
2. Verify volume health: `csinfo -H`
3. Check memory allocation: `confd_client db_info db_name: qa | grep mem`

---

## 9. Backup Configuration

### 9.1 Add Remote Archive Volume

**Purpose**: Configure remote storage (Azure, AWS S3, etc.) for backups.

#### Azure Blob Storage

```bash
confd_client remote_volume_add \
  remote_volume_name: "backup" \
  vol_type: azure \
  url: 'https://stexasolqaweu001.blob.core.windows.net/sc-exasol-qa-weu-001' \
  owner: '[500, 500]' \
  username: "stexasolqaweu001" \
  password: "YOUR_STORAGE_ACCESS_KEY"
```

#### AWS S3

```bash
confd_client remote_volume_add \
  remote_volume_name: "backup" \
  vol_type: s3 \
  url: 's3://bucket-name/path' \
  owner: '[500, 500]' \
  username: "AWS_ACCESS_KEY_ID" \
  password: "AWS_SECRET_ACCESS_KEY"
```

**Parameters**:
- `remote_volume_name`: Logical name for volume
- `vol_type`: Storage type (`azure`, `s3`, `gcs`)
- `url`: Full URL to storage container/bucket
- `owner`: UID and GID (`[500,500]` = exasol user)
- `username`: Storage account name or access key ID
- `password`: Storage access key or secret

**Verification**:

```bash
# List remote volumes
confd_client remote_volume_list

# Test remote volume access (SDFS)
sdfs list r0001
```

### 9.2 Database Restore

**Purpose**: Restore database from existing backup.

#### List Available Backups

```bash
# List all backups (including foreign)
confd_client db_backup_list db_name: qa show_foreign: True
```

**Expected Output**:
```yaml
- bid: 1
  comment: ''
  dependencies: '-'
  expire: ''
  expired: false
  id: 10002 EXA_VAL_QUAL/id_1/level_0/node_0/backup_202501271310 EXA_VAL_QUAL
  last_item: false
  level: 0
  path: EXA_VAL_QUAL/id_1/level_0/node_0/backup_202501271310
  system: EXA_VAL_QUAL
  timestamp: 2025-01-27 13:10
  ts: '202501271310'
  usable: true
  usage: 217.788 GiB
  volume: backupdev
```

**Backup Fields**:
- `id`: Full backup identifier (used for restore)
- `level`: Backup level (0=full, 1+=incremental)
- `timestamp`: Backup creation time
- `usable`: Whether backup can be restored
- `system`: Source database system ID
- `usage`: Backup size

#### Stop Database Before Restore

```bash
# Database must be stopped for restore
confd_client db_stop db_name: qa
```

#### Execute Restore

```bash
confd_client db_restore \
  db_name: qa \
  restore_type: blocking \
  backup_id: '10002 EXA_VAL_QUAL/id_1/level_0/node_0/backup_202501271310 EXA_VAL_QUAL'
```

**Parameters**:
- `db_name`: Target database
- `restore_type`: `blocking` (wait for completion) or `virtual` (background)
- `backup_id`: Full ID from `db_backup_list` output

#### Monitor Restore Progress

```bash
# Check restore progress
confd_client db_backup_progress db_name: qa
```

**Expected Output**:
```yaml
Comment: Restore is active
Files: []
Level: 0
Name: EXA_VAL_QUAL/id_1/level_0/node_0/backup_202501271310
Progress: 42
Type: Restore
Volume ID: 10002
```

**Progress Values**: 0-100 (percentage complete)

### 9.3 Create Backup Schedules

**Purpose**: Automate regular backups with retention policies.

#### Weekly Full Backup

```bash
confd_client db_backup_add_schedule \
  db_name: dev \
  backup_name: weekly_full_backup \
  backup_volume_id: 10001 \
  enabled: true \
  level: 0 \
  expire: "1w 3d" \
  minute: "0" \
  hour: "0" \
  weekday: "0"
```

**Schedule**: Every Sunday at 00:00, keep for 10 days

#### Daily Incremental Backup

```bash
confd_client db_backup_add_schedule \
  db_name: dev \
  backup_name: daily_incremental \
  backup_volume_id: 10001 \
  enabled: true \
  level: 1 \
  expire: '3d' \
  minute: 0 \
  hour: 0 \
  weekday: '1,2,3,4,5,6'
```

**Schedule**: Monday-Saturday at 00:00, keep for 3 days

**Parameters**:
- `backup_name`: Descriptive schedule name
- `backup_volume_id`: Volume ID from `remote_volume_list`
- `enabled`: `true` to activate schedule
- `level`: `0` (full), `1`+ (incremental)
- `expire`: Retention period (`3d`, `2w`, `1M`)
- `minute`, `hour`: Time of day (24-hour format)
- `weekday`: Days to run (`0`=Sunday, `1`=Monday, ..., `6`=Saturday)

**Best Practices**:
- **Full backups**: Weekly or monthly
- **Incremental backups**: Daily
- **Retention**: Match RPO/RTO requirements
- **Timing**: Schedule during low-activity periods

**Verify Schedules**:

```bash
# List backup schedules
confd_client db_backup_list_schedule db_name: dev
```

---

## 10. BucketFS Setup

### 10.1 Create BucketFS Instance

**Purpose**: Set up BucketFS for storing JDBC drivers, virtual schemas, UDF scripts.

```bash
# Create BucketFS instance
confd_client bucketfs_add \
  bucketfs_name: bucketfs1 \
  http_port: 0 \
  https_port: 2582 \
  owner: [500,500]
```

**Parameters**:
- `bucketfs_name`: BucketFS instance name
- `http_port`: HTTP port (use `0` to disable HTTP)
- `https_port`: HTTPS port (2581-2589 typical range)
- `owner`: UID and GID

**Security**: Always use HTTPS (`http_port: 0`) in production.

### 10.2 Create Buckets

**Purpose**: Create logical buckets within BucketFS for different content types.

```bash
# Virtual schema bucket
confd_client bucket_add \
  bucket_name: virtualschema \
  bucketfs_name: bucketfs1 \
  public: false \
  read_password: "YOUR_READ_PASSWORD" \
  write_password: "YOUR_WRITE_PASSWORD"

# JDBC drivers bucket
confd_client bucket_add \
  bucket_name: jdbcdrivers \
  bucketfs_name: bucketfs1 \
  public: false \
  read_password: "YOUR_READ_PASSWORD" \
  write_password: "YOUR_WRITE_PASSWORD"

# ETL jars bucket
confd_client bucket_add \
  bucket_name: etljars \
  bucketfs_name: bucketfs1 \
  public: false \
  read_password: "YOUR_READ_PASSWORD" \
  write_password: "YOUR_WRITE_PASSWORD"

# Schema mappings bucket
confd_client bucket_add \
  bucket_name: mappings \
  bucketfs_name: bucketfs1 \
  public: false \
  read_password: "YOUR_READ_PASSWORD" \
  write_password: "YOUR_WRITE_PASSWORD"
```

**Parameters**:
- `bucket_name`: Bucket name (alphanumeric)
- `bucketfs_name`: Parent BucketFS instance
- `public`: `true` (no auth) or `false` (password-protected)
- `read_password`: Password for read-only access
- `write_password`: Password for read/write access

**Security Best Practices**:
- Use strong passwords (16+ characters)
- Set `public: false` for sensitive content
- Use different passwords for read vs. write
- Rotate passwords regularly

### 10.3 Verify BucketFS Configuration

```bash
# List BucketFS instances
confd_client bucketfs_list

# Get detailed BucketFS info
confd_client bucketfs_info bucketfs_name: "bucketfs1"
```

**Expected Output**:
```yaml
_sec_name: 'BucketFS : bucketfs1'
buckets:
  etljars:
    _sec_name: 'Bucket : etljars'
    name: etljars
    public: false
    read_passwd: c29tZXBhc3N3b3JkMQ==
    write_passwd: c29tZXBhc3N3b3JkMQ==
  jdbcdrivers:
    _sec_name: 'Bucket : jdbcdrivers'
    name: jdbcdrivers
    public: false
    read_passwd: c29tZXBhc3N3b3JkMQ==
    write_passwd: c29tZXBhc3N3b3JkMQ==
  virtualschema:
    _sec_name: 'Bucket : virtualschema'
    name: virtualschema
    public: false
    read_passwd: c29tZXBhc3N3b3JkMQ==
    write_passwd: c29tZXBhc3N3b3JkMQ==
bucketvolume: None
http_port: 0
https_port: 2582
mode: rsync
name: bucketfs1
owner:
- 500
- 500
sync_period: '30000'
```

**Note**: Passwords are base64-encoded in output.

### 10.4 Upload Files to BucketFS

**Purpose**: Upload JDBC drivers, virtual schema JARs, and other artifacts.

#### Set Environment Variables

```bash
# Export bucket write password
export WRITE_PW="YOUR_WRITE_PASSWORD"

# Set BucketFS host (localhost if on data node, or node IP)
export BUCKETFS_HOST="localhost"
export BUCKETFS_PORT="2582"
```

#### Upload JDBC Drivers

**PostgreSQL Driver**:
```bash
# Upload settings configuration
curl -v -k -X PUT \
  -T drivers_jdbc_postgres_settings.cfg \
  https://w:$WRITE_PW@$BUCKETFS_HOST:2581/default/drivers/jdbc/postgres/settings.cfg

# Upload PostgreSQL JDBC driver
curl -v -k -X PUT \
  -T postgresql-42.2.24.jar \
  https://w:$WRITE_PW@$BUCKETFS_HOST:2581/default/drivers/jdbc/postgres/postgresql-42.2.24.jar
```

**Microsoft SQL Server Driver**:
```bash
curl -v -k -X PUT \
  -T drivers_jdbc_mssql_settings.cfg \
  https://w:$WRITE_PW@$BUCKETFS_HOST:2581/default/drivers/jdbc/mssql/settings.cfg

curl -v -k -X PUT \
  -T mssql-jdbc-12.8.1.jre11.jar \
  https://w:$WRITE_PW@$BUCKETFS_HOST:2581/default/drivers/jdbc/mssql/mssql-jdbc-12.8.1.jre11.jar
```

**Oracle Driver**:
```bash
curl -v -k -X PUT \
  -T drivers_jdbc_oracle_settings.cfg \
  https://w:$WRITE_PW@$BUCKETFS_HOST:2581/default/drivers/jdbc/oracle/settings.cfg

curl -v -k -X PUT \
  -T ojdbc11.jar \
  https://w:$WRITE_PW@$BUCKETFS_HOST:2581/default/drivers/jdbc/oracle/ojdbc11.jar
```

**SAP HANA Driver**:
```bash
curl -v -k -X PUT \
  -T drivers_jdbc_sap_settings.cfg \
  https://w:$WRITE_PW@$BUCKETFS_HOST:2581/default/drivers/jdbc/sap/settings.cfg

curl -v -k -X PUT \
  -T ngdbc.jar \
  https://w:$WRITE_PW@$BUCKETFS_HOST:2581/default/drivers/jdbc/sap/ngdbc.jar
```

**Databricks Driver**:
```bash
curl -v -k -X PUT \
  -T drivers_jdbc_databricks_settings.cfg \
  https://w:$WRITE_PW@$BUCKETFS_HOST:2581/default/drivers/jdbc/databricks/settings.cfg

curl -v -k -X PUT \
  -T DatabricksJDBC42.jar \
  https://w:$WRITE_PW@$BUCKETFS_HOST:2581/default/drivers/jdbc/databricks/DatabricksJDBC42.jar
```

#### Upload Virtual Schema Files

```bash
# Navigate to virtual schema directory
cd virtualschema

# Upload all virtual schema JARs
for file in *.jar; do
  curl -v -k -X PUT \
    -T "$file" \
    https://w:$WRITE_PW@$BUCKETFS_HOST:2582/virtualschema/
done
```

#### Upload ETL Jars

```bash
cd etljars

# Upload cloud storage extension
curl -v -k -X PUT \
  -T exasol-cloud-storage-extension-2.0.0.jar \
  https://w:$WRITE_PW@$BUCKETFS_HOST:2582/etljars/
```

#### Upload Schema Mappings

```bash
cd mappings

# Upload all mapping files
for file in *; do
  curl -v -k -X PUT \
    -T "$file" \
    https://w:$WRITE_PW@$BUCKETFS_HOST:2582/mappings/
done
```

#### Upload Batch Script

**Automated Upload Script**:
```bash
#!/bin/bash
# upload_bucketfs.sh

WRITE_PW="YOUR_WRITE_PASSWORD"
BUCKETFS_HOST="localhost"
DEFAULT_PORT="2581"
CUSTOM_PORT="2582"

# Function to upload file
upload_file() {
  local file=$1
  local bucket=$2
  local path=$3
  local port=$4
  
  echo "Uploading $file to $bucket/$path..."
  curl -k -X PUT \
    -T "$file" \
    "https://w:$WRITE_PW@$BUCKETFS_HOST:$port/$bucket/$path"
}

# Upload JDBC drivers
cd jdbc_drivers
for driver_dir in */; do
  for file in "$driver_dir"*; do
    upload_file "$file" "default" "drivers/jdbc/$driver_dir" "$DEFAULT_PORT"
  done
done

# Upload virtual schemas
cd ../virtualschema
for file in *.jar; do
  upload_file "$file" "virtualschema" "" "$CUSTOM_PORT"
done

echo "Upload complete!"
```

**Usage**:
```bash
chmod +x upload_bucketfs.sh
./upload_bucketfs.sh
```

#### Verify Uploads

```bash
# List bucket contents via HTTP
curl -k https://w:$WRITE_PW@localhost:2582/virtualschema/

# Or via BucketFS CLI (from COS)
bucketfs ls bucketfs1/virtualschema
```

---

## 11. Troubleshooting

### 11.1 Installation Issues

#### Issue: c4 Installation Fails

**Symptoms**: Installation stops at early stage, nodes don't reach stage 'd'.

**Diagnostic Steps**:
```bash
# Check c4 logs
./c4 logs

# SSH to node and check system logs
ssh adminuser@NODE_IP
sudo journalctl -u c4_cloud_command -f
```

**Common Causes**:
- **Incorrect IP addresses**: Verify `CCC_HOST_ADDRS` in c4-config
- **SSH key issues**: Ensure `CCC_HOST_KEY_PAIR_FILE` is correct
- **Disk problems**: Verify LVM volumes exist: `sudo lvdisplay`
- **Network issues**: Check connectivity between nodes

#### Issue: Nodes Stuck in Stage 'a' or 'b'

**Solution**:
```bash
# Check node status
./c4 ps

# View detailed logs on stuck node
ssh adminuser@NODE_IP
sudo journalctl -f | grep -i error
```

### 11.2 Database Issues

#### Issue: Database Won't Start

**Diagnostic Steps**:
```bash
# Check database info
confd_client db_info db_name: mydb

# Check logs
tail -f /exa/logs/cored/DataStorage.log
```

**Common Causes**:
- **Insufficient memory**: Reduce `mem_size` in db_create
- **Volume issues**: Verify volume exists and is online
- **Node problems**: Check node health with `cosps -N`

#### Issue: Database in 'failed' State

**Solution**:
```bash
# Stop database
confd_client db_stop db_name: mydb

# Check for errors in logs
grep -i error /exa/logs/cored/*

# Attempt to start again
confd_client db_start db_name: mydb
```

### 11.3 Storage Issues

#### Issue: Disk Shows 'OFFLINE'

**Diagnostic Steps**:
```bash
# Check disk health
csinfo -H

# Check system logs for disk errors
grep -i "I/O error" /var/log/syslog
```

**Solution**:
- Replace failed disk
- Contact Exasol support for data recovery

#### Issue: Volume Creation Fails

**Common Errors**:
- **"Not enough space"**: Reduce volume size or add more disks
- **"Invalid nodes"**: Verify node IDs are correct
- **"Disk not found"**: Check disk exists in `csinfo -D`

### 11.4 BucketFS Issues

#### Issue: File Upload Fails

**Diagnostic Steps**:
```bash
# Test bucket connectivity
curl -k https://w:PASSWORD@localhost:2582/bucketname/

# Check BucketFS logs
tail -f /exa/logs/cored/bucketfsd-*.log
```

**Common Causes**:
- **Wrong password**: Verify write password
- **SSL certificate**: Use `-k` flag with curl for self-signed certs
- **Port blocked**: Check firewall rules for HTTPS port

### 11.5 Network Issues

#### Issue: Cannot Connect to Database

**Diagnostic Steps**:
```bash
# Check database is running
confd_client db_info db_name: mydb | grep state

# Test port connectivity from client
telnet NODE_IP 8563

# Check firewall on node
sudo iptables -L -n | grep 8563
```

**Solution**:
- Open port 8563 in firewall
- Verify database is in 'running' state
- Check network routing

### 11.6 LDAP Authentication Issues

#### Issue: LDAP Users Cannot Login

**Diagnostic Steps**:
```bash
# Verify LDAP server is set
confd_client db_info db_name: mydb | grep ldap

# Test LDAP connectivity from node
ldapsearch -H ldaps://ads.example.com:636 -x -b "DC=example,DC=com"
```

**Common Causes**:
- **Wrong LDAP URL**: Verify `ldap_server` format
- **Certificate issues**: LDAPS requires valid SSL cert
- **Distinguished name incorrect**: Verify DN format in CREATE USER

**Solution**:
```bash
# Update LDAP server (requires database restart)
confd_client db_stop db_name: mydb
confd_client db_configure db_name: mydb ldap_server: 'ldaps://correct-server:636'
confd_client db_start db_name: mydb
```

---

## Summary

This guide covered:
- ✅ SSH access to Exasol data nodes
- ✅ LVM storage preparation
- ✅ File transfer and c4 configuration
- ✅ Complete Exasol installation process
- ✅ System health validation
- ✅ License management
- ✅ Database creation and configuration
- ✅ Backup and restore procedures
- ✅ BucketFS setup and file uploads
- ✅ Common troubleshooting scenarios

**Next Steps**:
- Configure monitoring (Telegraf, Grafana)
- Set up user accounts and permissions
- Create schemas and tables
- Configure virtual schemas for external data sources
- Implement backup verification procedures

**Related Documentation**:
- [c4 Comprehensive Guide](c4_comprehensive_guide.md)
- [Exasol Node Management](exasol_node_management.md)
- [Exasol Storage Management](exasol_storage_management.md)
- [Exasol Update Procedure](exasol_update_procedure.md)
- [Exasol Directory Structure](exasol_directory_structure.md)

---

## Document Metadata

- **Created**: 2025-02-02
- **Exasol Version**: 8.32.0
- **c4 Version**: Latest
- **Tested Environment**: On-premise and cloud deployments
- **Categories**: Installation, Configuration, Database Management, Storage, Backup
