# Exasol Rootless Deployment (Non-Root Installation)

**Category:** Administration  
**Topic:** Installation, Security, Configuration, Deployment, User Management  
**Keywords:** rootless, non-root, security, subuid, subgid, preplay, c4, user namespaces, huge pages, privileges, permissions  
**Source:** [Exasol Rootless Deployment](https://docs.exasol.com/db/latest/administration/on-premise/installation/install_rootless.htm)

## Overview

This document describes additional configuration steps required when deploying Exasol with a **non-root user** (rootless deployment). This enhanced security approach allows Exasol to run without requiring root privileges on database hosts.

**What is Rootless Deployment?**
- Exasol runs under a **non-root user** account
- Enhanced security through privilege separation
- Additional system configuration required
- Uses Linux user namespaces and subuid/subgid mappings

**Use Cases:**
- High-security environments
- Compliance requirements (regulatory, security policies)
- Organizations requiring non-root service execution
- Environments with strict privilege separation

**Important**: This document covers **only the additional steps** for rootless deployment. For the complete installation procedure, see [Install Exasol - Step by Step](https://docs.exasol.com/db/latest/administration/on-premise/installation/install_hw.htm).

---

## Prerequisites

### Standard Requirements

All standard installation prerequisites still apply:
- Supported Linux distribution (Ubuntu 20.04/22.04/24.04, RHEL 8/9/10)
- Hardware meeting minimum requirements
- Network configuration
- Storage devices prepared

**See also**: [System Requirements](https://docs.exasol.com/db/latest/administration/on-premise/installation/system_requirements.htm)

### Additional Requirements for Rootless

**System Packages:**
- `newuidmap` and `newgidmap` tools must be installed

**User Configuration:**
- Non-root Exasol user created on all nodes
- Subuid/subgid ranges allocated (minimum 60,000 IDs)
- User has specific capabilities for container creation

**Root/Sudo Access:**
- Root or sudo access required for:
  - Initial system configuration
  - Running preplay script
  - Configuring disk permissions
  - Setting system parameters

---

## Rootless Deployment Overview

### Workflow

```
┌──────────────────────────────────────────────────────────┐
│              ROOTLESS DEPLOYMENT STEPS                   │
├──────────────────────────────────────────────────────────┤
│ 1. Allocate subuids and subgids            [10 min]     │
│ 2. Install required tools                  [5 min]      │
│ 3. Run preplay script                      [10 min]     │
│ 4. Configure data disks                    [15 min]     │
│ 5. Configure huge pages                    [10 min]     │
│ 6. Add rootless parameter to config        [5 min]      │
│ 7. Proceed with standard installation      [varies]     │
└──────────────────────────────────────────────────────────┘

Total additional time: ~1 hour (compared to standard install)
```

### Architecture Comparison

**Standard (Root) Installation:**
```
┌─────────────────────────────────┐
│         Host System             │
│  ┌───────────────────────────┐  │
│  │  Exasol (runs as root)    │  │
│  │  ├─ COS Container          │  │
│  │  ├─ Database Services      │  │
│  │  └─ Storage Management     │  │
│  └───────────────────────────┘  │
└─────────────────────────────────┘
```

**Rootless Installation:**
```
┌─────────────────────────────────┐
│         Host System             │
│  ┌───────────────────────────┐  │
│  │  Exasol User (non-root)   │  │
│  │  ├─ User Namespaces        │  │
│  │  │  └─ COS Container       │  │
│  │  ├─ Subuid/Subgid Mapping  │  │
│  │  └─ Database Services      │  │
│  └───────────────────────────┘  │
└─────────────────────────────────┘
```

---

## Step 1: Allocate Subuids and Subgids

**Objective**: Allocate subordinate user and group IDs to enable the non-root Exasol user to create and run containers.

### Understanding Subuids/Subgids

**What are they?**
- **Subuid**: Subordinate user IDs
- **Subgid**: Subordinate group IDs
- Allow non-root users to map UID/GID ranges in user namespaces
- Enable rootless container execution

**Requirements:**
- Minimum **60,000** subuids allocated to Exasol user
- Minimum **60,000** subgids allocated to Exasol user
- Ranges must be **identical** on all hosts
- Ranges must **not overlap** with other users

### Configure Subuid/Subgid Files

**Files to edit:**
- `/etc/subuid` - Subordinate user IDs
- `/etc/subgid` - Subordinate group IDs

**Format:**
```
username:start_id:count
```

**Example allocation:**

```bash
# Edit /etc/subuid
sudo nano /etc/subuid
```

**Add line for Exasol user:**
```
exasol:100000:65535
```

**Edit /etc/subgid:**
```bash
sudo nano /etc/subgid
```

**Add line for Exasol user:**
```
exasol:100000:65535
```

**Explanation:**
- `exasol`: Username of installation user
- `100000`: Starting subordinate ID
- `65535`: Number of IDs allocated (meets minimum of 60,000)

### Apply on All Nodes

**Critical**: Configuration must be **identical** on all database nodes.

```bash
# Node 1 (n0011)
echo "exasol:100000:65535" | sudo tee -a /etc/subuid
echo "exasol:100000:65535" | sudo tee -a /etc/subgid

# Node 2 (n0012)
ssh n0012
echo "exasol:100000:65535" | sudo tee -a /etc/subuid
echo "exasol:100000:65535" | sudo tee -a /etc/subgid
exit

# Repeat for all nodes...
```

### Verify Configuration

```bash
# Check subuid allocation
grep exasol /etc/subuid
# Output: exasol:100000:65535

# Check subgid allocation
grep exasol /etc/subgid
# Output: exasol:100000:65535

# Verify on all nodes
for node in n0011 n0012 n0013 n0014; do
  echo "=== $node ==="
  ssh $node "grep exasol /etc/subuid"
  ssh $node "grep exasol /etc/subgid"
done
```

**Important**: All nodes must show identical output.

---

## Step 2: Install Required Tools

**Objective**: Install `newuidmap` and `newgidmap` utilities required for user namespace mapping.

### Ubuntu Installation

```bash
# Install uidmap package
sudo apt update
sudo apt install uidmap

# Verify installation
which newuidmap newgidmap
# Output:
# /usr/bin/newuidmap
# /usr/bin/newgidmap
```

### RHEL/CentOS Installation

```bash
# Tools pre-installed on RHEL 8 and later
# Verify they exist
which newuidmap newgidmap

# If missing, install shadow-utils
sudo yum install shadow-utils
```

### Verify Installation on All Nodes

```bash
# Check all nodes
for node in n0011 n0012 n0013 n0014; do
  echo "=== $node ==="
  ssh $node "which newuidmap newgidmap"
done
```

**For more information**: See Linux man pages for `subuid`, `subgid`, `newuidmap`, and `newgidmap`.

---

## Step 3: Run the Preplay Script

**Objective**: Automatically apply most rootless configuration requirements using the embedded preplay script.

### What the Preplay Script Does

The preplay script (embedded in c4) automatically configures:

1. **Enable unprivileged user namespaces** (if disabled)
   - Required to create COS container namespaces

2. **Increase resource limits** for Exasol user
   - Number of open files
   - Number of processes
   - Stack size
   - Amount of locked memory

3. **Disable transparent huge pages**
   - Prevents problems with sparse memory mappings

4. **Apply sysctl settings**
   - Performance and stability optimizations

### Run Preplay Script

**Execute on each database node:**

```bash
# Switch to root or use sudo
sudo ./c4 _ preplay exasol
```

**Replace** `exasol` with your Exasol username if different.

**Important Notes:**
- Must be run **as root or with sudo**
- If logged in as root, omit `sudo`
- Must be run **on each database node**
- Run before starting installation

### Run on All Nodes

**From jump host or first node:**

```bash
# Copy c4 binary to each node
for node in n0011 n0012 n0013 n0014; do
  scp c4 exasol@$node:~/
done

# Run preplay on each node
for node in n0011 n0012 n0013 n0014; do
  echo "=== Running preplay on $node ==="
  ssh $node "sudo ./c4 _ preplay exasol"
done
```

**Alternative: Log in to each node individually:**

```bash
# Node 1
ssh n0011
sudo ./c4 _ preplay exasol
exit

# Node 2
ssh n0012
sudo ./c4 _ preplay exasol
exit

# Repeat for all nodes...
```

### View Script Contents (Optional)

**To see what the preplay script does without executing:**

```bash
./c4 _ preplay -D
```

**Example output:**
```bash
#!/bin/bash
# Preplay script for rootless Exasol deployment

# Enable unprivileged user namespaces
echo 1 > /proc/sys/kernel/unprivileged_userns_clone

# Increase resource limits
cat > /etc/security/limits.d/exasol.conf << EOF
exasol soft nofile 1048576
exasol hard nofile 1048576
exasol soft nproc  131072
exasol hard nproc  131072
exasol soft stack  unlimited
exasol hard stack  unlimited
exasol soft memlock unlimited
exasol hard memlock unlimited
EOF

# Disable transparent huge pages
echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo never > /sys/kernel/mm/transparent_hugepage/defrag

# Apply sysctl settings
cat > /etc/sysctl.d/90-exasol.conf << EOF
vm.max_map_count = 2147483647
kernel.shmmax = 549755813888
kernel.shmall = 134217728
...
EOF

sysctl --system
```

### Verify Preplay Execution

```bash
# Check user limits
ssh n0011 "su - exasol -c 'ulimit -n'"
# Should show: 1048576

# Check userns enabled
ssh n0011 "cat /proc/sys/kernel/unprivileged_userns_clone"
# Should show: 1

# Check transparent huge pages disabled
ssh n0011 "cat /sys/kernel/mm/transparent_hugepage/enabled"
# Should show: always ... [never]
```

---

## Step 4: Configure Data Disks

**Objective**: Configure disk permissions so the non-root Exasol user can read/write to storage devices.

### Why Disk Configuration is Needed

- Storage devices (`/dev/sdb`, `/dev/nvme1n1`, etc.) are typically owned by root
- Non-root Exasol user needs read/write access
- Use **udev rules** to set persistent ownership
- Ensures permissions persist across reboots

### Configuration Methods

Choose the appropriate method based on your storage setup:
1. **Physical disks**: Direct block devices
2. **Logical volumes (LVM)**: LVM-managed storage

---

### Method 1: Physical Disk Setup

**Use this method for direct block devices** (e.g., `/dev/nvme1n1`, `/dev/sdb`).

#### Identify Exasol User UID

```bash
# Get Exasol user UID
id -u exasol
# Example output: 1001

# Save to variable
exasol_uid=$(id -u exasol)
echo $exasol_uid
```

#### Identify Storage Disks

```bash
# List all block devices
lsblk

# Example output:
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda           8:0    0   100G  0 disk 
├─sda1        8:1    0    99G  0 part /
└─sda2        8:2    0     1G  0 part [SWAP]
nvme1n1     259:0    0   1.5T  0 disk 
nvme2n1     259:1    0   1.5T  0 disk 

# nvme1n1 = data volume
# nvme2n1 = archive volume
```

#### Create udev Rules

**Script to create udev rules:**

```bash
#!/bin/bash
# Create udev rules for Exasol disk ownership

exasol_uid=1001  # Replace with your Exasol user UID
disks=(/dev/nvme1n1 /dev/nvme2n1)  # Replace with your disks

for disk in "${disks[@]}"; do
  disk_basename="$(basename "$(readlink -f "$disk")")"
  owner="$(getent passwd "$exasol_uid" | cut -d: -f1)"
  echo 'KERNEL=="'"$disk_basename"'", OWNER="'"$owner"'"' \
    >> /etc/udev/rules.d/90-exasol.rules
done
```

**Execute as root:**

```bash
sudo bash -c '
exasol_uid=1001
disks=(/dev/nvme1n1 /dev/nvme2n1)

for disk in "${disks[@]}"; do
  disk_basename="$(basename "$(readlink -f "$disk")")"
  owner="$(getent passwd "$exasol_uid" | cut -d: -f1)"
  echo "KERNEL==\"$disk_basename\", OWNER=\"$owner\"" \
    >> /etc/udev/rules.d/90-exasol.rules
done
'
```

#### Verify udev Rules File

```bash
# Check created rules
cat /etc/udev/rules.d/90-exasol.rules

# Example output:
KERNEL=="nvme1n1", OWNER="exasol"
KERNEL=="nvme2n1", OWNER="exasol"
```

#### Apply udev Rules

```bash
# Reload udev rules and trigger
sudo udevadm control --reload-rules && sudo udevadm trigger

# Verify disk ownership
ls -l /dev/nvme1n1 /dev/nvme2n1

# Should show exasol as owner:
# brw-rw---- 1 exasol disk 259, 0 Feb  6 10:00 /dev/nvme1n1
# brw-rw---- 1 exasol disk 259, 1 Feb  6 10:00 /dev/nvme2n1
```

#### Apply to All Nodes

```bash
# Create udev rules on all nodes
for node in n0011 n0012 n0013 n0014; do
  echo "=== Configuring $node ==="
  ssh $node "sudo bash -c '
    exasol_uid=1001
    disks=(/dev/nvme1n1 /dev/nvme2n1)
    
    for disk in \"\${disks[@]}\"; do
      disk_basename=\"\$(basename \"\$(readlink -f \"\$disk\")\")\"
      owner=\"\$(getent passwd \"\$exasol_uid\" | cut -d: -f1)\"
      echo \"KERNEL==\\\"\$disk_basename\\\", OWNER=\\\"\$owner\\\"\" \
        >> /etc/udev/rules.d/90-exasol.rules
    done
    
    udevadm control --reload-rules && udevadm trigger
  '"
done
```

---

### Method 2: Logical Volume Setup (LVM)

**Use this method for LVM-managed storage** (e.g., `/dev/mapper/vg_name/lv_name`).

#### Identify Logical Volumes

```bash
# List logical volumes
sudo lvs

# Example output:
LV        VG       Attr       LSize   Pool Origin Data%  Meta%
lv_data   vg_exasol -wi-a-----   1.00t
lv_archive vg_exasol -wi-a-----   1.00t

# Paths:
# /dev/mapper/vg_exasol/lv_data
# /dev/mapper/vg_exasol/lv_archive
```

#### Create udev Rules for LVM

**Create `/etc/udev/rules.d/90-exasol.rules`:**

```bash
sudo cat > /etc/udev/rules.d/90-exasol.rules << 'EOF'
SUBSYSTEM=="block", ENV{DM_VG_NAME}=="vg_exasol", ENV{DM_LV_NAME}=="lv_data", OWNER="exasol", MODE="0660"
SUBSYSTEM=="block", ENV{DM_VG_NAME}=="vg_exasol", ENV{DM_LV_NAME}=="lv_archive", OWNER="exasol", MODE="0660"
EOF
```

**Adjust for your volume group and logical volume names.**

#### Apply udev Rules

```bash
# Reload and trigger udev
sudo udevadm control --reload-rules && sudo udevadm trigger

# Verify ownership
ls -l /dev/mapper/vg_exasol/lv_*

# Should show exasol as owner:
# brw-rw---- 1 exasol disk 253, 2 Feb  6 10:00 /dev/mapper/vg_exasol/lv_data
# brw-rw---- 1 exasol disk 253, 3 Feb  6 10:00 /dev/mapper/vg_exasol/lv_archive
```

#### Apply to All Nodes

```bash
# Distribute udev rules to all nodes
for node in n0011 n0012 n0013 n0014; do
  echo "=== Configuring $node ==="
  scp /etc/udev/rules.d/90-exasol.rules root@$node:/etc/udev/rules.d/
  ssh $node "sudo udevadm control --reload-rules && sudo udevadm trigger"
  ssh $node "ls -l /dev/mapper/vg_exasol/lv_*"
done
```

---

## Step 5: Configure Huge Pages

**Objective**: Configure huge pages memory allocation for database performance and stability.

### Understanding Huge Pages

**Why huge pages are required:**
- Improves memory management performance
- Reduces page table overhead
- Provides more stable memory allocation
- Critical for database performance

**Configuration requires:**
- Calculate number of huge pages needed
- Set `vm.nr_hugepages` sysctl parameter
- Set `vm.hugetlb_shm_group` to allow Exasol user access

### Calculate Huge Pages

**Formula:**
```
nr_hugepages = ((DB RAM / number of nodes) - maxSystemHeapMemory) / huge_page_size
```

**Where:**
- `DB RAM`: Total memory allocated to database across cluster
- `number of nodes`: Number of database nodes in cluster
- `maxSystemHeapMemory`: Default **32 GiB**
- `huge_page_size`: Typically **2 MiB** (check with `grep Hugepagesize /proc/meminfo`)

**Example calculation:**
```
Database RAM: 256 GB
Nodes: 4
Max system heap: 32 GB

Per-node memory: 256 GB / 4 = 64 GB
Available for huge pages: 64 GB - 32 GB = 32 GB
Number of pages: 32 GB / 2 MiB = 32768 MiB / 2 MiB = 16384 pages
```

### Determine Hugetlb Shared Memory Group

**Formula:**
```
vm.hugetlb_shm_group = subgid_start + 55553
```

**Where:**
- `subgid_start`: Starting subordinate GID from `/etc/subgid`

**Example:**
```bash
# Get subgid start for exasol user
grep exasol /etc/subgid
# Output: exasol:100000:65535

subgid_start = 100000
vm.hugetlb_shm_group = 100000 + 55553 = 155553
```

### Configuration Script

```bash
#!/bin/bash
# Configure huge pages for Exasol rootless deployment

EXASOL_USER=exasol
NR_HUGEPAGES=16384  # Adjust based on your calculation

# Get subgid start for Exasol user
GROUP_ID_SUBGID=$(grep -w "$EXASOL_USER" /etc/subgid | awk -F ":" '{ print $2 }')

# Calculate hugetlb_shm_group
HUGETLB_SHM_GROUP=$((GROUP_ID_SUBGID + 55553))

# Write parameters to sysctl conf file
echo -e "vm.nr_hugepages=$NR_HUGEPAGES\nvm.hugetlb_shm_group=$HUGETLB_SHM_GROUP" \
  | sudo tee /etc/sysctl.d/92-exasol-hugepages.conf

# Apply the new parameters
sudo sysctl --system

# Verify
echo "=== Verification ==="
sysctl vm.nr_hugepages
sysctl vm.hugetlb_shm_group
```

### Execute Configuration

```bash
# Copy script to file
cat > configure_hugepages.sh << 'EOF'
#!/bin/bash
EXASOL_USER=exasol
NR_HUGEPAGES=16384

GROUP_ID_SUBGID=$(grep -w "$EXASOL_USER" /etc/subgid | awk -F ":" '{ print $2 }')
HUGETLB_SHM_GROUP=$((GROUP_ID_SUBGID + 55553))

echo -e "vm.nr_hugepages=$NR_HUGEPAGES\nvm.hugetlb_shm_group=$HUGETLB_SHM_GROUP" \
  | sudo tee /etc/sysctl.d/92-exasol-hugepages.conf

sudo sysctl --system

echo "=== Verification ==="
sysctl vm.nr_hugepages
sysctl vm.hugetlb_shm_group
EOF

chmod +x configure_hugepages.sh

# Run on each node
for node in n0011 n0012 n0013 n0014; do
  echo "=== Configuring huge pages on $node ==="
  scp configure_hugepages.sh $node:~/
  ssh $node "./configure_hugepages.sh"
done
```

### Verify Configuration

```bash
# Check huge pages configuration
grep Huge /proc/meminfo

# Example output:
AnonHugePages:         0 kB
ShmemHugePages:        0 kB
FileHugePages:         0 kB
HugePages_Total:   16384
HugePages_Free:    16384
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB

# Check sysctl parameters
sysctl vm.nr_hugepages
sysctl vm.hugetlb_shm_group
```

---

## Step 6: Add Rootless Configuration Parameter

**Objective**: Enable rootless mode in c4 configuration.

### Option 1: Add to Configuration File

**Edit your c4 configuration file:**

```yaml
# config.yaml
version: 1
cluster:
  name: exasol-cluster

# Add rootless parameter
CCC_PLAY_ROOTLESS: true

nodes:
  - name: n0011
    ...
```

**Alternatively, add as top-level parameter:**

```yaml
version: 1

# Rootless deployment mode
rootless: true

cluster:
  name: exasol-cluster
  ...
```

### Option 2: Add to Command Line

**Include parameter when running c4 play:**

```bash
./c4 --ccc-play-rootless true host play -i config.yaml
```

### Verification

**Check configuration is recognized:**

```bash
# Dry run to verify
./c4 --ccc-play-rootless true host play -i config.yaml --dry-run
```

---

## Step 7: Proceed with Installation

**Continue with standard installation procedure** as described in the main installation guide.

**Installation command:**

```bash
# From jump host
./c4 --ccc-play-rootless true host play -i config.yaml

# Or if rootless parameter is in config file
./c4 host play -i config.yaml
```

**See**: [Install Exasol - Step by Step](https://docs.exasol.com/db/latest/administration/on-premise/installation/install_hw.htm)

---

## Post-Installation: Rootless-Specific Operations

### Interacting with c4_cloud_command Service

**In rootless mode**, all systemd services are **user services** belonging to the Exasol user, not system services.

**Key Differences:**
- Do **not** use `sudo` with service commands
- Use `systemctl --user` instead of `systemctl`
- Services run under Exasol user context

### Check Service Status

```bash
# As Exasol user (not root)
su - exasol

# Check c4_cloud_command service status
systemctl --user status c4_cloud_command

# Example output:
● c4_cloud_command.service - Exasol Cloud Command Service
     Loaded: loaded (/home/exasol/.config/systemd/user/c4_cloud_command.service)
     Active: active (running) since Thu 2026-02-06 10:00:00 UTC; 1h ago
   Main PID: 12345 (c4_cloud_command)
     ...
```

### View Service Logs

```bash
# As Exasol user
journalctl --user -u c4_cloud_command

# Follow logs in real-time
journalctl --user -u c4_cloud_command -f

# Show last 100 lines
journalctl --user -u c4_cloud_command -n 100
```

### Start/Stop/Restart Services

```bash
# As Exasol user
systemctl --user start c4_cloud_command
systemctl --user stop c4_cloud_command
systemctl --user restart c4_cloud_command

# Enable/disable service
systemctl --user enable c4_cloud_command
systemctl --user disable c4_cloud_command
```

### XDG_RUNTIME_DIR Variable

**Some systems require XDG_RUNTIME_DIR to be set:**

```bash
# If you get errors about XDG_RUNTIME_DIR, export it:
export XDG_RUNTIME_DIR=/run/user/$(id -u "$USER")

# Make persistent in .bashrc
echo 'export XDG_RUNTIME_DIR=/run/user/$(id -u "$USER")' >> ~/.bashrc
```

---

## Troubleshooting

### Issue: Subuid/Subgid Not Configured Correctly

**Error message:**
```
Error: Unable to create user namespace
```

**Diagnosis:**
```bash
# Check subuid/subgid allocation
grep exasol /etc/subuid
grep exasol /etc/subgid

# Verify counts are sufficient (minimum 60,000)
```

**Solution:**
```bash
# Edit files and add/correct entries
sudo nano /etc/subuid
sudo nano /etc/subgid

# Ensure format: exasol:100000:65535
# Verify on all nodes
```

### Issue: newuidmap/newgidmap Not Found

**Error message:**
```
Error: newuidmap: command not found
```

**Solution:**
```bash
# Ubuntu
sudo apt install uidmap

# RHEL (usually pre-installed)
sudo yum install shadow-utils

# Verify
which newuidmap newgidmap
```

### Issue: Disk Permission Denied

**Error message:**
```
Error: Permission denied accessing /dev/nvme1n1
```

**Diagnosis:**
```bash
# Check disk ownership
ls -l /dev/nvme1n1

# Should show exasol as owner
```

**Solution:**
```bash
# Recreate udev rules
sudo vim /etc/udev/rules.d/90-exasol.rules

# Reload rules
sudo udevadm control --reload-rules && sudo udevadm trigger

# Verify ownership
ls -l /dev/nvme1n1
```

### Issue: Huge Pages Not Allocated

**Error message:**
```
Warning: Unable to allocate huge pages
```

**Diagnosis:**
```bash
# Check huge pages
grep HugePages /proc/meminfo

# Check sysctl parameters
sysctl vm.nr_hugepages
sysctl vm.hugetlb_shm_group
```

**Solution:**
```bash
# Recalculate and set parameters
sudo vi /etc/sysctl.d/92-exasol-hugepages.conf

# Apply
sudo sysctl --system

# Verify
grep HugePages /proc/meminfo
```

### Issue: Service Commands Fail

**Error message:**
```
Failed to connect to bus: No such file or directory
```

**Solution:**
```bash
# Export XDG_RUNTIME_DIR
export XDG_RUNTIME_DIR=/run/user/$(id -u)

# Retry command
systemctl --user status c4_cloud_command

# Make permanent
echo 'export XDG_RUNTIME_DIR=/run/user/$(id -u)' >> ~/.bashrc
```

### Issue: Database Performance Issues

**Symptoms:**
- Slow query performance
- High memory swapping
- System instability

**Diagnosis:**
```bash
# Check huge pages usage
grep Huge /proc/meminfo

# Check if transparent huge pages disabled
cat /sys/kernel/mm/transparent_hugepage/enabled
# Should show: [never]
```

**Solution:**
- Verify huge pages configured correctly
- Ensure transparent huge pages disabled (preplay script does this)
- Re-run preplay script if needed

---

## Quick Reference

### Rootless Configuration Checklist

**Pre-installation:**
- [ ] Subuid/subgid allocated (minimum 60,000) in `/etc/subuid` and `/etc/subgid`
- [ ] Ranges identical on all nodes
- [ ] newuidmap/newgidmap installed
- [ ] Preplay script executed on all nodes
- [ ] Disk permissions configured (udev rules)
- [ ] Huge pages configured
- [ ] Rootless parameter added to config

**Verification:**
- [ ] `grep exasol /etc/subuid` shows allocation
- [ ] `which newuidmap` returns path
- [ ] `ls -l /dev/nvme1n1` shows exasol owner
- [ ] `grep HugePages /proc/meminfo` shows allocation
- [ ] `sysctl vm.hugetlb_shm_group` shows correct value

### Essential Commands

```bash
# Check subuid/subgid
grep exasol /etc/subuid /etc/subgid

# Run preplay script
sudo ./c4 _ preplay exasol

# Check disk ownership
ls -l /dev/nvme1n1 /dev/nvme2n1

# Check huge pages
grep HugePages /proc/meminfo
sysctl vm.nr_hugepages vm.hugetlb_shm_group

# Service management (as Exasol user)
systemctl --user status c4_cloud_command
journalctl --user -u c4_cloud_command

# Install with rootless mode
./c4 --ccc-play-rootless true host play -i config.yaml
```

### Configuration File Template

```yaml
version: 1

# Enable rootless deployment
rootless: true

cluster:
  name: exasol-cluster
image: "@exasol-8.34.0"

nodes:
  - name: n0011
    private_ip: 10.0.1.11
    ssh:
      host: n0011
      user: exasol  # Non-root user
      private_key_file: ~/.ssh/id_rsa
    disks:
      - name: data
        devices: ["/dev/nvme1n1"]
      - name: archive
        devices: ["/dev/nvme2n1"]
  # ... additional nodes
```

---

## Related Documentation

- [Install Exasol - Step by Step](https://docs.exasol.com/db/latest/administration/on-premise/installation/install_hw.htm)
- [System Requirements](https://docs.exasol.com/db/latest/administration/on-premise/installation/system_requirements.htm)
- [Exasol Deployment Tool (c4)](https://docs.exasol.com/db/latest/administration/on-premise/admin_interface/c4.htm)
- [How to Use c4](https://docs.exasol.com/db/latest/administration/on-premise/c4/using_c4.htm)
- [Prepare Storage Devices](https://docs.exasol.com/db/latest/administration/on-premise/installation/install_as_app/configure_storage.htm)

## Common Questions

- What is rootless Exasol deployment?
- Why should I use rootless installation?
- What are subuid and subgid?
- How do I allocate subordinate user IDs?
- What is the preplay script and what does it do?
- How do I configure disk permissions for rootless installation?
- How do I configure huge pages for rootless Exasol?
- What is the difference between root and rootless installation?
- Can I convert existing root installation to rootless?
- How do I manage systemd services in rootless mode?
- Why do I get "permission denied" for disk access?
- How do I troubleshoot XDG_RUNTIME_DIR errors?
- What are user namespaces?
- How many subordinate IDs do I need?
- Do subuid/subgid ranges need to be identical on all nodes?
- Can I use different ranges for different users?
- How do I verify rootless installation is working correctly?

## Summary

Rootless Exasol deployment involves:
- **Enhanced security**: Non-root user execution
- **Additional configuration**: Subuid/subgid, preplay script, disk permissions, huge pages
- **User namespaces**: Enable container execution without root
- **Systemd user services**: Services run under Exasol user context

**Key steps:**
1. Allocate subuids/subgids (minimum 60,000, identical on all nodes)
2. Install newuidmap/newgidmap tools
3. Run preplay script on all nodes (as root/sudo)
4. Configure disk permissions with udev rules
5. Configure huge pages (vm.nr_hugepages, vm.hugetlb_shm_group)
6. Add rootless parameter to configuration
7. Proceed with standard installation

**Critical requirements:**
- Subuid/subgid ranges:**minimum 60,000**, identical on **all nodes**
- Disk ownership: Exasol user must own storage devices
- Huge pages: Properly calculated and configured
- Preplay script: Executed on **all nodes**

**Service management:**
- Use `systemctl --user` instead of `systemctl`
- Do **not** use sudo for service commands
- Export XDG_RUNTIME_DIR if needed

**For help**: [Create a support case](https://exasol.my.site.com/s/create-new-case)
