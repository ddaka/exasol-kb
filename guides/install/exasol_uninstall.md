# Exasol Uninstall Procedure (On-Premise)

**Category:** Administration  
**Topic:** Uninstall, Removal, Cleanup, Maintenance, Decommissioning  
**Keywords:** uninstall, remove, delete, cleanup, decommission, c4, systemd, services, directories  
**Source:** [Exasol Uninstall Documentation](https://docs.exasol.com/db/latest/administration/on-premise/installation/uninstall.htm)

## Overview

This document explains how to completely uninstall Exasol from a host system, including removing all services, files, directories, and configurations created during installation.

**Use Cases:**
- Decommissioning an Exasol cluster
- Complete removal before reinstallation
- Cleaning up failed installation attempts
- Repurposing hardware for other uses

**Important Warnings:**
- ⚠️ **Uninstallation is irreversible** - All data will be permanently deleted
- ⚠️ **Create backups before uninstalling** if you need to preserve data
- ⚠️ **Perform on all nodes** in the cluster for complete removal
- ⚠️ **Requires root or sudo access** on all hosts

---

## Prerequisites

### Access Requirements

**Required Access:**
- Root user access **OR**
- Non-root user with sudo privileges
- SSH access to all database nodes

**Verification:**
```bash
# Check sudo access
sudo whoami
# Output: root

# Verify access to all nodes
for node in n0011 n0012 n0013 n0014; do
  ssh $node "sudo whoami"
done
```

### Tools Requirements

**Required Tools:**
- `c4` command-line tool (for connecting to cluster)
- `confd_client` (for database operations)
- SSH client

**Tool Documentation:**
- [Exasol Deployment Tool (c4)](https://docs.exasol.com/db/latest/administration/on-premise/admin_interface/c4.htm)
- [ConfD](https://docs.exasol.com/db/latest/confd/confd.htm)
- [How to Use c4](https://docs.exasol.com/db/latest/administration/on-premise/c4/using_c4.htm)

---

## Backup Considerations

### Before You Begin: Create Backups

**If you need to preserve any data, create backups BEFORE uninstalling:**

#### 1. Database Backup

```bash
# Get Play ID
PLAY_ID=$(./c4 ps | awk 'NR==2 {print $3}')

# Connect to COS
./c4 connect -i $PLAY_ID -s cos

# Create backup
confd_client db_backup \
  db_name: MY_DATABASE \
  volume_name: DataVolume1 \
  backup_dir: "s3://my-bucket/final-backup/"

# Verify backup completed
confd_client db_backup_list db_name: MY_DATABASE
```

#### 2. BucketFS Backup

```bash
# Download all BucketFS files
# Method 1: Using BucketFS Client
exaplus -c "http://n0011:2580/default" -u w:write_password \
  -command "EXPORT * TO LOCAL CSV FILE 'bucketfs_backup/'"

# Method 2: Manual download
wget -r --no-parent http://n0011:2580/default/

# Save to safe location
tar -czf bucketfs_backup.tar.gz bucketfs_backup/
```

#### 3. Configuration Backup

```bash
# Save c4 configuration file
cp config.yaml config-backup-$(date +%Y%m%d).yaml

# Export cluster information
./c4 ps > cluster-info-$(date +%Y%m%d).txt
./c4 connect -i $PLAY_ID -s cos -- 'confd_client db_info db_name: MY_DATABASE' \
  > db-info-$(date +%Y%m%d).txt

# Save to safe location
tar -czf exasol-config-backup-$(date +%Y%m%d).tar.gz *.yaml *.txt
```

**See also**: [Backup and Restore](https://docs.exasol.com/db/latest/administration/on-premise/backup_restore.htm)

---

## Uninstall Procedure

### Overview

The uninstall procedure consists of two main steps:
1. **Shut down the database** - Stop database services gracefully
2. **Remove resources on hosts** - Clean up files, services, and directories

**Estimated time**: 30-60 minutes (varies with cluster size)

---

## Step 1: Shut Down the Database

**Objective**: Gracefully stop the database before removing system resources.

### 1.1 Get Play ID

```bash
# List all deployments
./c4 ps

# Example output:
  N  PLAY_ID   NODE  MEDIUM  INSTANCE     DB_VERSION  EXTERNAL_IP     INTERNAL_IP  STAGE  STATE    UPTIME    TTL
┌─  1  c3275f84  11    hwcf    bare-metal   8.34.0      203.0.113.11    10.0.1.11    d      running  03:50:12  +∞
│   1  c3275f84  12    hwcf    bare-metal   8.34.0      203.0.113.12    10.0.1.12    d      running  03:50:13  +∞
│   1  c3275f84  13    hwcf    bare-metal   8.34.0      203.0.113.13    10.0.1.13    d      running  03:50:13  +∞
└─  1  c3275f84  14    hwcf    bare-metal   8.34.0      203.0.113.14    10.0.1.14    d      running  03:50:13  +∞

# Save Play ID
PLAY_ID=c3275f84
```

### 1.2 Connect to Cluster Operating System

```bash
# Connect to COS
./c4 connect -i $PLAY_ID -s cos

# You should see a shell prompt in the COS environment
```

**See also**: [How to Use c4 - Connect to Deployment](https://docs.exasol.com/db/latest/administration/on-premise/c4/using_c4.htm#Connecttoadeployment)

### 1.3 Check Database State

```bash
# Check current database state
confd_client db_state db_name: MY_DATABASE

# Possible outputs:
# - "running": Database is active
# - "setup": Database is stopped
# - "stopping": Database is shutting down
```

**See also**: [db_state ConfD Job](https://docs.exasol.com/db/latest/confd/jobs/db_state.htm)

### 1.4 Stop the Database (if Running)

```bash
# Stop database
confd_client db_stop db_name: MY_DATABASE

# Wait for shutdown to complete (may take 1-5 minutes)
# Monitor status
while true; do
  state=$(confd_client db_state db_name: MY_DATABASE)
  echo "Database state: $state"
  [ "$state" == "setup" ] && break
  sleep 5
done

echo "Database stopped successfully"
```

**Expected output:**
```
Result:
'setup'
```

**See also**: [db_stop ConfD Job](https://docs.exasol.com/db/latest/confd/jobs/db_stop.htm)

### 1.5 Exit COS Connection

```bash
# Exit from COS shell
exit

# You should return to your normal shell prompt
```

---

## Step 2: Remove Resources on Hosts

**Objective**: Remove all Exasol-related services, files, and directories from each host.

**Important**: Procedures differ slightly between **root** and **rootless** installations.

---

### Method A: Standard (Root) Installation Removal

**Use this method if Exasol was installed with root privileges (default).**

#### 2.1 Stop and Disable System Services

**Execute on EACH database node:**

```bash
# Stop c4 services
sudo systemctl stop c4_cloud_command
sudo systemctl stop c4

# Disable services (prevent auto-start)
sudo systemctl disable c4_cloud_command
sudo systemctl disable c4
```

**Verify services stopped:**
```bash
sudo systemctl status c4_cloud_command
sudo systemctl status c4

# Should show: "inactive (dead)"
```

#### 2.2 Remove Service Files

```bash
# Remove c4 service files
sudo rm /etc/systemd/system/c4_cloud_command.service
sudo rm /etc/systemd/system/c4_cloud_command.env
sudo rm /etc/systemd/system/c4.service
sudo rm /etc/systemd/system/c4.env
```

#### 2.3 Stop and Remove Admin UI Service (Exasol 2025.1+)

**Only if using Exasol 2025.1 or later:**

```bash
# Stop and disable Admin UI
sudo systemctl stop exasol-admin-ui
sudo systemctl disable exasol-admin-ui

# Remove service files
sudo rm /etc/systemd/system/exasol-admin-ui.service
sudo rm /etc/systemd/system/exasol-admin-ui.env
```

**Note**: Skip this step for Exasol versions before 2025.1.

#### 2.4 Remove c4 Directories and Files

```bash
# Remove c4 directories
sudo rm -r /var/lib/ccc
sudo rm -r /usr/local/bin/c4
sudo rm -r /var/log/ccc
```

#### 2.5 Unmount and Remove /x Mount Point

```bash
# Unmount /x (if mounted)
sudo umount /x

# Remove mount point
sudo rm -rf /x
```

**Verify unmounted:**
```bash
mount | grep /x
# Should return nothing
```

#### 2.6 Remove Exasol User Home Directory

```bash
# Remove .ccc directory from Exasol user home
# If logged in as Exasol user:
sudo rm -r ~/.ccc

# If logged in as root:
sudo rm -r /home/exasol/.ccc
```

#### 2.7 Remove SSH Keys

```bash
# Remove c4 SSH keys
# If logged in as Exasol user:
sudo rm -rf ~/.ssh/c4-*

# If logged in as root:
sudo rm -rf /home/exasol/.ssh/c4-*
```

#### 2.8 Reload systemd

```bash
# Reload systemd to recognize removed services
sudo systemctl daemon-reload
```

---

### Method B: Rootless Installation Removal

**Use this method if Exasol was installed with a non-root user.**

#### 2.1 Stop and Disable User Services

**Execute as the Exasol user (not root):**

```bash
# Switch to Exasol user
su - exasol

# Stop c4 services
systemctl --user stop c4_cloud_command
systemctl --user stop c4

# Disable services
systemctl --user disable c4_cloud_command
systemctl --user disable c4
```

**Verify services stopped:**
```bash
systemctl --user status c4_cloud_command
systemctl --user status c4

# Should show: "inactive (dead)"
```

#### 2.2 Remove User Service Files

```bash
# As Exasol user
rm $HOME/.config/systemd/user/c4_cloud_command.service
rm $HOME/.config/systemd/user/c4_cloud_command.env
rm $HOME/.config/systemd/user/c4.service
rm $HOME/.config/systemd/user/c4.env
```

#### 2.3 Stop and Remove Admin UI Service (Exasol 2025.1+)

**Only if using Exasol 2025.1 or later:**

```bash
# As Exasol user
systemctl --user stop exasol-admin-ui
systemctl --user disable exasol-admin-ui

# Remove service files
rm $HOME/.config/systemd/user/exasol-admin-ui.service
rm $HOME/.config/systemd/user/exasol-admin-ui.env
```

#### 2.4 Remove c4 Binary

```bash
# As Exasol user
rm $HOME/.local/bin/c4
```

#### 2.5 Remove Exasol User Home Directory

```bash
# Exit from Exasol user
exit

# As root or with sudo
sudo rm -r /home/exasol/.ccc
```

#### 2.6 Remove SSH Keys

```bash
# As root or with sudo
sudo rm -rf /home/exasol/.ssh/c4-*
```

#### 2.7 Reload User systemd

```bash
# As Exasol user
su - exasol
systemctl --user daemon-reload
exit
```

---

### 2.8 Restart Host (Recommended)

**After completing removal on each node:**

```bash
# Restart to reset all temporary system parameters
sudo reboot
```

**Why restart is recommended:**
- Resets temporary system parameters
- Clears memory allocations (huge pages)
- Ensures complete cleanup of processes
- Required for full parameter reset

**Alternative (if reboot not possible):**
```bash
# Reset sysctl parameters manually
sudo sysctl --system

# Clear huge pages
echo 0 | sudo tee /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages
```

---

## Step 3: Repeat on All Nodes

**Critical**: Perform the removal procedure on **every database node** in the cluster.

### Automated Removal Script

**For faster removal across multiple nodes:**

```bash
#!/bin/bash
# Exasol Uninstall Script (root installation)

NODES="n0011 n0012 n0013 n0014"

for node in $NODES; do
  echo "=== Removing Exasol from $node ==="
  
  ssh $node "sudo bash -s" << 'EOF'
    # Stop services
    systemctl stop c4_cloud_command c4 2>/dev/null
    systemctl disable c4_cloud_command c4 2>/dev/null
    
    # Stop Admin UI (Exasol 2025.1+)
    systemctl stop exasol-admin-ui 2>/dev/null
    systemctl disable exasol-admin-ui 2>/dev/null
    
    # Remove service files
    rm -f /etc/systemd/system/c4_cloud_command.*
    rm -f /etc/systemd/system/c4.*
    rm -f /etc/systemd/system/exasol-admin-ui.*
    
    # Remove directories
    rm -rf /var/lib/ccc
    rm -rf /usr/local/bin/c4
    rm -rf /var/log/ccc
    
    # Unmount /x
    umount /x 2>/dev/null
    rm -rf /x
    
    # Remove .ccc directory
    rm -rf /home/exasol/.ccc
    rm -rf /home/exasol/.ssh/c4-*
    
    # Reload systemd
    systemctl daemon-reload
    
    echo "Cleanup complete on $(hostname)"
EOF

  echo "=== $node cleanup complete ==="
  echo ""
done

echo "=== All nodes cleaned up ==="
echo "Consider rebooting all nodes to complete removal"
```

**Save and execute:**
```bash
chmod +x uninstall_exasol.sh
./uninstall_exasol.sh
```

---

## Post-Uninstall Cleanup

### Remove Exasol User (Optional)

**If the Exasol user is no longer needed:**

```bash
# On each node (must be logged in as different user with sudo)
sudo userdel -r exasol

# Verify removal
id exasol
# Should show: "no such user"

# Remove any remaining user files
sudo rm -rf /home/exasol
```

**WARNING**: Only remove the Exasol user if you're certain it won't be needed.

### Remove udev Rules (Optional)

**If you configured custom udev rules for disk ownership:**

```bash
# Remove Exasol udev rules
sudo rm /etc/udev/rules.d/90-exasol.rules

# Reload udev
sudo udevadm control --reload-rules && sudo udevadm trigger
```

### Remove sysctl Configuration (Optional)

**Remove Exasol-specific sysctl configurations:**

```bash
# Remove sysctl files
sudo rm /etc/sysctl.d/90-exasol.conf
sudo rm /etc/sysctl.d/91-exasol-apparmor-userns.conf
sudo rm /etc/sysctl.d/92-exasol-hugepages.conf

# Reload sysctl
sudo sysctl --system
```

### Remove Security Limits (Optional)

**Remove resource limits for Exasol user:**

```bash
# Remove limits file
sudo rm /etc/security/limits.d/exasol.conf
```

### Clean Storage Devices (Optional)

**If you want to completely wipe storage devices:**

**⚠️ WARNING: This PERMANENTLY DESTROYS all data on the disks!**

```bash
# Wipe disk with zeros
sudo dd if=/dev/zero of=/dev/nvme1n1 bs=1M count=1000 status=progress

# Or use wipefs
sudo wipefs -a /dev/nvme1n1
sudo wipefs -a /dev/nvme2n1

# Verify
sudo lsblk -f
```

---

## Verification

### Verify Complete Removal

**Check that all components are removed:**

```bash
#!/bin/bash
# Exasol Removal Verification Script

echo "=== Exasol Removal Verification ==="

# Check services
echo "1. Checking services..."
if systemctl list-unit-files | grep -q c4; then
    echo "   [WARN] c4 services still present"
else
    echo "   [OK] No c4 services found"
fi

# Check directories
echo "2. Checking directories..."
[ -d /var/lib/ccc ] && echo "   [WARN] /var/lib/ccc exists" || echo "   [OK] /var/lib/ccc removed"
[ -d /var/log/ccc ] && echo "   [WARN] /var/log/ccc exists" || echo "   [OK] /var/log/ccc removed"
[ -d /x ] && echo "   [WARN] /x exists" || echo "   [OK] /x removed"

# Check binaries
echo "3. Checking binaries..."
[ -f /usr/local/bin/c4 ] && echo "   [WARN] c4 binary exists" || echo "   [OK] c4 binary removed"

# Check user home
echo "4. Checking user directories..."
[ -d /home/exasol/.ccc ] && echo "   [WARN] .ccc exists" || echo "   [OK] .ccc removed"

# Check processes
echo "5. Checking processes..."
if ps aux | grep -v grep | grep -q c4_cloud_command; then
    echo "   [WARN] c4_cloud_command process still running"
else
    echo "   [OK] No c4 processes running"
fi

echo "=== Verification Complete ==="
```

**Run on each node:**
```bash
for node in n0011 n0012 n0013 n0014; do
  echo "=== Verifying $node ==="
  ssh $node "bash -s" < verify_removal.sh
done
```

---

## Troubleshooting

### Issue: Services Won't Stop

**Error:**
```
Failed to stop c4_cloud_command.service: Unit c4_cloud_command.service not loaded
```

**Solution:**
```bash
# Check if service exists
systemctl list-unit-files | grep c4

# If exists but won't stop, kill processes
sudo pkill -9 c4_cloud_command

# Then remove service files
sudo rm /etc/systemd/system/c4_cloud_command.*
sudo systemctl daemon-reload
```

### Issue: /x Won't Unmount

**Error:**
```
umount: /x: target is busy
```

**Solution:**
```bash
# Check what's using /x
sudo lsof +D /x
sudo fuser -vm /x

# Kill processes using /x
sudo fuser -km /x

# Force unmount
sudo umount -f /x

# Or lazy unmount (detaches when no longer busy)
sudo umount -l /x
```

### Issue: Permission Denied Removing Directories

**Error:**
```
rm: cannot remove '/var/lib/ccc': Permission denied
```

**Solution:**
```bash
# Use sudo
sudo rm -rf /var/lib/ccc

# If still fails, check file attributes
sudo lsattr /var/lib/ccc

# Remove immutable attribute if set
sudo chattr -i /var/lib/ccc/*
sudo rm -rf /var/lib/ccc
```

### Issue: Database Won't Stop

**Error:**
```
Database state: stopping (stuck)
```

**Solution:**
```bash
# Check database processes
./c4 connect -i $PLAY_ID -s cos -- 'ps aux | grep exasol'

# Force stop via ConfD
confd_client db_stop db_name: MY_DATABASE force: true

# If still stuck, restart c4_cloud_command service
sudo systemctl restart c4_cloud_command

# Or force kill processes
sudo pkill -9 -f exasol
```

---

## Reinstallation After Uninstall

**If you plan to reinstall Exasol:**

### Preparation Checklist

- [ ] All nodes restarted after uninstall
- [ ] Storage devices cleaned or prepared
- [ ] Network configuration verified
- [ ] Exasol user recreated (if removed)
- [ ] SSH keys regenerated
- [ ] System requirements verified

### Reinstallation Steps

1. **Verify cleanup complete** (use verification script)
2. **Prepare hosts** (network, storage, users)
3. **Download latest c4** version
4. **Create new configuration file**
5. **Run installation procedure**

**See**: [Install Exasol - Step by Step](https://docs.exasol.com/db/latest/administration/on-premise/installation/install_hw.htm)

---

## Quick Reference

### Standard (Root) Removal Commands

```bash
# Stop services
sudo systemctl stop c4_cloud_command c4 exasol-admin-ui
sudo systemctl disable c4_cloud_command c4 exasol-admin-ui

# Remove service files
sudo rm /etc/systemd/system/c4_cloud_command.*
sudo rm /etc/systemd/system/c4.*
sudo rm /etc/systemd/system/exasol-admin-ui.*

# Remove directories
sudo rm -rf /var/lib/ccc /usr/local/bin/c4 /var/log/ccc
sudo umount /x && sudo rm -rf /x
sudo rm -rf /home/exasol/.ccc /home/exasol/.ssh/c4-*

# Reload systemd
sudo systemctl daemon-reload

# Reboot
sudo reboot
```

### Rootless Removal Commands

```bash
# As Exasol user
systemctl --user stop c4_cloud_command c4 exasol-admin-ui
systemctl --user disable c4_cloud_command c4 exasol-admin-ui

# Remove service files (as Exasol user)
rm $HOME/.config/systemd/user/c4_cloud_command.*
rm $HOME/.config/systemd/user/c4.*
rm $HOME/.config/systemd/user/exasol-admin-ui.*
rm $HOME/.local/bin/c4

# Exit and remove as root
exit
sudo rm -rf /home/exasol/.ccc /home/exasol/.ssh/c4-*

# Reboot
sudo reboot
```

### Verification Commands

```bash
# Check services removed
systemctl list-unit-files | grep c4

# Check directories removed
ls -la /var/lib/ccc /var/log/ccc /x /home/exasol/.ccc

# Check processes
ps aux | grep c4_cloud_command

# Check mount points
mount | grep /x
```

---

## Related Documentation

- [Install Exasol - Step by Step](https://docs.exasol.com/db/latest/administration/on-premise/installation/install_hw.htm)
- [Exasol Deployment Tool (c4)](https://docs.exasol.com/db/latest/administration/on-premise/admin_interface/c4.htm)
- [How to Use c4](https://docs.exasol.com/db/latest/administration/on-premise/c4/using_c4.htm)
- [ConfD](https://docs.exasol.com/db/latest/confd/confd.htm)
- [Rootless Deployment](https://docs.exasol.com/db/latest/administration/on-premise/installation/install_rootless.htm)
- [Backup and Restore](https://docs.exasol.com/db/latest/administration/on-premise/backup_restore.htm)
- [db_stop ConfD Job](https://docs.exasol.com/db/latest/confd/jobs/db_stop.htm)
- [db_state ConfD Job](https://docs.exasol.com/db/latest/confd/jobs/db_state.htm)

## Common Questions

- How do I uninstall Exasol completely?
- What should I backup before uninstalling Exasol?
- How do I remove Exasol services?
- What directories does Exasol create?
- How do I stop the Exasol database before uninstalling?
- Can I uninstall Exasol without losing data?
- What's the difference between root and rootless uninstall?
- How do I remove Exasol from multiple nodes?
- Do I need to restart after uninstalling Exasol?
- How do I verify Exasol is completely removed?
- Can I reinstall Exasol after uninstalling?
- How do I remove the Exasol user?
- What happens to storage devices after uninstall?
- How do I clean up failed Exasol installation?
- How do I unmount /x if it's busy?
- How do I force stop Exasol services?
- What files does Exasol create during installation?
- How do I remove exasol-admin-ui service?
- Do I need to remove sysctl configurations?
- How do I wipe Exasol storage devices?

## Summary

Uninstalling Exasol involves:
- **Step 1**: Shut down database gracefully using `db_stop`
- **Step 2**: Remove services, directories, and files on all nodes
- **Step 3**: Restart hosts to reset system parameters

**Complete removal checklist:**
- [ ] Database stopped
- [ ] Services stopped and disabled
- [ ] Service files removed
- [ ] Directories removed (/var/lib/ccc, /var/log/ccc, /x, ~/.ccc)
- [ ] Binaries removed (/usr/local/bin/c4)  
- [ ] SSH keys removed (~/.ssh/c4-*)
- [ ] systemd reloaded
- [ ] Hosts restarted

**Root vs Rootless differences:**
- **Root**: Use `systemctl` without --user flag, remove from /etc/systemd/system/
- **Rootless**: Use `systemctl --user` as Exasol user, remove from ~/.config/systemd/user/

**Critical steps:**
1. Create backups if data preservation needed
2. Stop database before removing services
3. Perform on **all nodes** in cluster
4. Restart hosts after removal
5. Verify complete cleanup

**Optional cleanup:**
- Remove Exasol user
- Remove udev rules
- Remove sysctl configurations
- Wipe storage devices

**For help**: [Create a support case](https://exasol.my.site.com/s/create-new-case)
