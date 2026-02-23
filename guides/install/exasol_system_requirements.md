# Exasol System Requirements (On-Premise)

**Category:** Administration  
**Topic:** Installation, Planning, Hardware, Infrastructure, System Configuration  
**Keywords:** system requirements, hardware, CPU, RAM, memory, storage, network, Linux, Ubuntu, RHEL, operating system, specifications, minimum, recommended  
**Source:** [Exasol System Requirements](https://docs.exasol.com/db/latest/administration/on-premise/installation/system_requirements.htm)

## Overview

This document describes the hardware, network, and operating system requirements for deploying Exasol database in on-premise environments. These requirements apply to both physical hardware and virtual machine deployments (AWS, Azure, Google Cloud).

**Scope:**
- Exasol data nodes (database servers)
- Optional jump host (installation system)
- Standard and rootless deployments
- GPU-enabled configurations (separate requirements)

**Important**: Meeting these requirements is **mandatory** for successful installation and optimal performance. Failure to meet minimum requirements may result in installation failures, performance degradation, or system instability.

---

## Hardware Requirements

### CPU and Firmware

**Processor Requirements:**
- **Architecture**: 64-bit x86 platforms
- **Intel CPUs**: SSSE3 featured processors (1 or 2 sockets)
- **AMD CPUs**: Single socket only
- **BIOS/UEFI**: Configured for maximum performance (recommended)

**Minimum Configuration:**
- **Development/Testing**: 4 cores per node
- **Production**: 16+ cores per node

**Recommended Configuration:**
- **Production**: 32+ cores per node
- **Large deployments**: 64+ cores per node

**Performance Notes:**
- Exasol is CPU-intensive for query processing
- More cores = better parallel query execution
- Hyper-threading supported and recommended

### Memory (RAM)

**Minimum Configuration:**
- **Development/Testing**: 16 GB per node
- **Production**: 64 GB per node

**Recommended Configuration:**
- **Production**: 128+ GB per node
- **Large deployments**: 256+ GB per node
- **Data-intensive workloads**: 512+ GB per node

**Memory Allocation Guidelines:**

| Data Size | RAM per Node | Cluster Size |
|-----------|--------------|--------------|
| **< 1 TB** | 64 GB | 4 nodes |
| **1-10 TB** | 128 GB | 4-8 nodes |
| **10-50 TB** | 256 GB | 8-16 nodes |
| **50-100 TB** | 512 GB | 16-32 nodes |
| **> 100 TB** | 1+ TB | 32+ nodes |

**Memory Considerations:**
- Database uses in-memory processing for optimal performance
- More RAM = larger datasets fit in memory
- Plan for data growth over time
- Reserve memory for OS and system processes

**See also**: [Sizing Guidelines](https://docs.exasol.com/db/latest/administration/on-premise/sizing.htm)

### Storage

#### Supported Storage Types

**Block Devices:**
- Local storage: SAS, SATA, SSD, NVMe
- Virtual disks (VMware, KVM, AWS EBS, Azure Disks)
- Remote storage: iSCSI, SAN
- LVM2 (Logical Volume Manager 2)
- LUKS (encrypted volumes)

**Filesystem-based:**
- Sparse file devices hosted on ext4 or XFS
- **Not supported**: NFS

**Strongly Recommended:**
- **SSD or NVMe** for data volumes (production)
- **Separate physical disks** for data and archive volumes
- **RAID 1** or similar fault tolerance for OS disks
- **LVM2** for persistent device naming

#### Storage Capacity Requirements

**Minimum Configuration:**
- **OS partition**: 150 GiB free space after installation
- **Data volumes**: 100 GB per node
- **Archive volumes**: 100 GB per node (for backups)

**Recommended Configuration:**
- **OS partition**: 200+ GiB
- **Data volumes**: 500+ GB per node (SSD/NVMe)
- **Archive volumes**: Equal to or larger than data volumes

**Production Guidelines:**

| Data Size | Data Volume per Node | Archive Volume per Node |
|-----------|---------------------|------------------------|
| **< 1 TB** | 250 GB | 250 GB |
| **1-10 TB** | 1 TB | 1 TB |
| **10-50 TB** | 5 TB | 5 TB |
| **50-100 TB** | 10 TB | 10 TB |
| **> 100 TB** | 20+ TB | 20+ TB |

#### Storage Configuration Requirements

**Disk Configuration:**
- Use **at least 4 storage drives** with minimum **250 MBps read/write capacity** per drive
- OS and storage disks should have **RAID 1 or similar fault tolerance**
- Swap partitions: Use size recommended by OS vendor

**Block Device Naming:**
- Block storage devices **must have persistent names**
- **Strongly recommend**: LVM2 for persistent device naming
- **Do not use**: `/dev/nvme1n1`, `/dev/sda`, `/dev/sdb` (not persistent)
- **If not using LVM**: Specify explicit path from `/dev/disk/by-id/`

**Example - Persistent Device Paths:**
```bash
# Good (persistent):
/dev/disk/by-id/nvme-Dell_Express_Flash_NVMe_PM1725b_1.6TB_123456789
/dev/mapper/vg_exasol/lv_data
/dev/mapper/vg_exasol/lv_archive

# Bad (not persistent):
/dev/nvme1n1
/dev/sdb
/dev/sdc
```

**See also**: [Prepare Storage Devices](https://docs.exasol.com/db/latest/administration/on-premise/installation/install_as_app/configure_storage.htm)

---

## Network Requirements

### Network Interfaces

**Supported Speeds:**
- **10 Gbps** (minimum for production)
- **25 Gbps** (recommended)
- **50 Gbps** (large deployments)
- **100 Gbps** (very large deployments)

**Configuration:**
- Database hosts should be assigned **static IPv4 addresses** in the same subnet
- DHCP can be used if each host **always receives the same IP address**

**Example IP Configuration:**
```
Private network (cluster communication):
10.0.1.11, 10.0.1.12, 10.0.1.13, 10.0.1.14

Public network (client access - optional):
203.0.113.11, 203.0.113.12, 203.0.113.13, 203.0.113.14
```

### Network Architecture

**Recommended Setup:**

```
                    Internet/Clients
                           │
                    ┌──────┴──────┐
                    │   Firewall   │
                    └──────┬──────┘
                           │
        ┌──────────────────┼──────────────────┐
        │         Public Network (1 Gbps)     │
        │         203.0.113.0/24              │
        └──────────────────┬──────────────────┘
                           │
        ┌────────┬─────────┼─────────┬────────┐
        │        │         │         │        │
    ┌───▼───┐ ┌─▼────┐ ┌──▼───┐ ┌──▼───┐    │
    │ Node1 │ │ Node2│ │ Node3│ │ Node4│    │
    │ .11   │ │ .12  │ │ .13  │ │ .14  │    │
    └───┬───┘ └──┬───┘ └──┬───┘ └──┬───┘    │
        │        │         │         │        │
        └────────┴─────────┴─────────┴────────┘
        │  Private Network (10+ Gbps)        │
        │  10.0.1.0/24                        │
        └────────────────────────────────────┘
```

### Required Ports

**Essential Ports:**

| Port | Protocol | Purpose | Direction | Required |
|------|----------|---------|-----------|----------|
| **22** | TCP | SSH access | Inbound/Outbound | Yes |
| **443** | TCP | Exasol Admin UI (HTTPS) | Inbound | Yes (2025.1+) |
| **8563** | TCP | Database connections | Inbound | Yes |
| **10000-10999** | TCP | Internal cluster communication | Between nodes | Yes |

**Optional Ports:**

| Port | Protocol | Purpose | Direction | Required |
|------|----------|---------|-----------|----------|
| **2580** | TCP | BucketFS HTTPS | Inbound | Optional |
| **20000** | TCP | BucketFS HTTP | Inbound | Optional |

**Firewall Configuration:**
- Internal cluster communication (ports 10000-10999) must be **unrestricted** between nodes
- External access should be **restricted** to necessary ports only
- Use **private IPs** for cluster communication
- Use **public IPs** for client connections (if separate network)

### Time Synchronization

**Required**: Network Time Protocol (NTP) must be configured and active on all nodes.

**Why Critical:**
- Cluster coordination requires synchronized time
- Log correlation depends on accurate timestamps
- Transaction consistency requires time sync

**Verification:**
```bash
# Check NTP status
timedatectl status

# Verify NTP is active
systemctl status chronyd  # RHEL/CentOS
systemctl status systemd-timesyncd  # Ubuntu

# Check time difference between nodes
for node in n0011 n0012 n0013 n0014; do
  echo "$node: $(ssh $node date)"
done
```

---

## Operating System Requirements

### Supported Linux Distributions

**Current Support Matrix:**

| Distribution | Supported Versions | Exasol Versions |
|--------------|-------------------|-----------------|
| **Ubuntu** | 24.04 LTS | 8.29, 8.34, 2025.1+ |
| **Ubuntu** | 22.04 LTS | All supported releases |
| **Ubuntu** | 20.04 LTS | All supported releases |
| **Red Hat Enterprise Linux** | 10 | 2025.2+ |
| **Red Hat Enterprise Linux** | 9 | All supported releases |
| **Red Hat Enterprise Linux** | 8 | All supported releases |

**Important Notes:**
- Exasol contains a full user-level runtime environment with minimal dependencies on host OS
- Future major kernel releases may affect Exasol environment
- Test thoroughly after OS updates
- Compatibility cannot be guaranteed across major OS updates

**See also**: [Release Notes](https://docs.exasol.com/db/latest/release_notes.htm)

### OS Configuration Requirements

**Required Configuration:**

#### 1. Firewall Configuration
```bash
# Option 1: Disable firewall (simplest)
sudo systemctl stop firewalld  # RHEL/CentOS
sudo systemctl disable firewalld

sudo systemctl stop ufw  # Ubuntu
sudo systemctl disable ufw

# Option 2: Configure firewall rules (recommended for security)
# See Network Requirements section for required ports
```

#### 2. Antivirus Scanners
```bash
# Disable antivirus scanners
# Antivirus can interfere with database operations
# Add Exasol directories to antivirus exclusions or disable
```

#### 3. SELinux Configuration
```bash
# If SELinux is installed, set to permissive mode
sudo setenforce 0

# Make persistent
sudo vi /etc/selinux/config
# Set: SELINUX=permissive

# Verify
getenforce
# Output: Permissive
```

#### 4. NTP Configuration
```bash
# RHEL/CentOS
sudo systemctl enable chronyd
sudo systemctl start chronyd

# Ubuntu
sudo systemctl enable systemd-timesyncd
sudo systemctl start systemd-timesyncd

# Verify
timedatectl status
```

#### 5. RHEL 8 and RHEL 9 - iptables Service
```bash
# Install and enable iptables service (required)
sudo yum install iptables-services
sudo systemctl enable iptables
sudo systemctl start iptables
```

#### 6. RHEL 8 - Python 3 Installation
```bash
# Install Python 3 (required on RHEL 8)
sudo dnf install python3

# Verify
python3 --version
```

#### 7. User Shell Configuration
```bash
# Login shell for Exasol user must be bash
sudo chsh -s $(which bash) exasol

# Verify
grep exasol /etc/passwd
# Should show: /bin/bash
```

#### 8. Umask Configuration
```bash
# Both Exasol user and root must have umask 0022 or less

# Check current umask
umask
# Should be: 0022 or lower (e.g., 0002)

# Set permanently in /etc/profile or ~/.bashrc
echo "umask 0022" >> ~/.bashrc
```

### AppArmor Configuration (Ubuntu 22.04, 24.04)

**Required for Ubuntu 22.04 and 24.04 only.**

**Kernel Parameter:**
- `kernel.apparmor_restrict_unprivileged_userns` must be **disabled**

**Automatic Configuration:**
- **Exasol 8.34.0 and later** (c4 version 4.25.0+): Automatically disabled during installation
- **No action required** for 8.34.0+

**Manual Configuration (Exasol 8.33.0 and earlier on Ubuntu 22.04):**

```bash
# Disable AppArmor parameter
echo 'kernel.apparmor_restrict_unprivileged_userns = 0' | \
  sudo tee /etc/sysctl.d/91-exasol-apparmor-userns.conf

# Apply immediately
sudo sysctl -q --system

# Verify
sudo sysctl kernel.apparmor_restrict_unprivileged_userns
# Output: kernel.apparmor_restrict_unprivileged_userns = 0
```

**Important**: This requirement does **not apply** to:
- Exasol 8.34.0 and later (handled automatically)
- Ubuntu 20.04 (AppArmor parameter not present)

---

## Jump Host Requirements

**What is a Jump Host?**
- External system used to run installation process
- Accesses database nodes over SSH
- Not part of the database cluster

**Jump Host Requirements:**

### Software Requirements
- **SSH client**: Installed and configured
- **SSH key-based authentication**: Configured to all database nodes
- **rsync**: Required for c4 version 4.24.3 or earlier (optional for 4.25.0+)

### Network Requirements
- **Network access**: Must be able to reach all database nodes
- **Port 22**: SSH access to all nodes
- **Stable connection**: Reliable network during installation

### System Requirements
- **OS**: Linux (any distribution), macOS, or WSL on Windows
- **Disk space**: 50+ GB for download and temporary files
- **RAM**: 4+ GB

**Jump Host Installation Commands:**

```bash
# Install SSH (usually pre-installed)
# Ubuntu/Debian
sudo apt install openssh-client

# RHEL/CentOS
sudo yum install openssh-clients

# Install rsync (if using c4 4.24.3 or earlier)
# Ubuntu/Debian
sudo apt install rsync

# RHEL/CentOS
sudo yum install rsync

# Generate SSH key for Exasol installation
ssh-keygen -t rsa -b 4096 -C "exasol-install"

# Copy SSH key to all database nodes
ssh-copy-id exasol@n0011
ssh-copy-id exasol@n0012
ssh-copy-id exasol@n0013
ssh-copy-id exasol@n0014

# Test connectivity
for node in n0011 n0012 n0013 n0014; do
  ssh exasol@$node hostname
done
```

---

## GPU Support Requirements

**For GPU-accelerated UDF processing:**

Special requirements apply when Exasol should support GPU acceleration for User-Defined Functions (UDFs).

**Additional Hardware:**
- NVIDIA GPU with CUDA support
- Adequate GPU memory (8+ GB recommended)
- PCIe connectivity

**Additional Software:**
- NVIDIA GPU drivers
- CUDA toolkit
- Additional configuration steps

**See also**: [System Requirements for GPU Support](https://docs.exasol.com/db/latest/administration/on-premise/gpu_support/gpu_support_system_requirements.htm), [GPU Support Installation](https://docs.exasol.com/db/latest/administration/on-premise/gpu_support.htm)

---

## Configuration Checklist

### Pre-Installation Verification

**Hardware Checklist:**
- [ ] CPUs meet minimum requirements (64-bit x86, SSSE3 support)
- [ ] Sufficient RAM (16+ GB minimum, 64+ GB recommended)
- [ ] Storage devices identified and available
- [ ] RAID configured for OS disks
- [ ] Network interfaces configured (10+ Gbps recommended)
- [ ] All nodes have identical hardware specifications (recommended)

**Operating System Checklist:**
- [ ] Supported Linux distribution installed
- [ ] Firewall disabled or configured
- [ ] Antivirus disabled or Exasol directories excluded
- [ ] SELinux in permissive mode (if installed)
- [ ] NTP configured and active on all nodes
- [ ] iptables service enabled (RHEL 8/9)
- [ ] Python 3 installed (RHEL 8)
- [ ] Exasol user shell set to bash
- [ ] Umask set to 0022 or less
- [ ] AppArmor parameter disabled (Ubuntu 22.04/24.04, Exasol < 8.34.0)

**Network Checklist:**
- [ ] Static IP addresses assigned to all nodes
- [ ] Hostname resolution working (DNS or /etc/hosts)
- [ ] Required ports open between nodes
- [ ] Firewall rules configured for external access
- [ ] Time synchronized across all nodes (NTP)
- [ ] Network speed meets minimum (10 Gbps for production)

**Storage Checklist:**
- [ ] Block devices identified (use `lsblk`)
- [ ] Device paths are persistent (LVM or /dev/disk/by-id/)
- [ ] Sufficient capacity for data and archive volumes
- [ ] OS partition has 150+ GiB free space
- [ ] Devices are raw (no existing filesystems)
- [ ] Permissions configured for Exasol user

**User Configuration Checklist:**
- [ ] Exasol user created on all nodes
- [ ] Exasol user has sudo privileges (standard) or proper subuid/subgid (rootless)
- [ ] SSH key-based authentication configured
- [ ] Passwordless SSH access verified to all nodes
- [ ] Passwordless sudo access verified

---

## Sizing Guidelines

### Cluster Sizing Examples

**Small Deployment (Development/Testing):**
- **Nodes**: 1-4
- **CPU per node**: 4-8 cores
- **RAM per node**: 16-32 GB
- **Storage per node**: 100-250 GB
- **Network**: 1-10 Gbps
- **Use case**: Development, testing, POC
- **Data capacity**: < 1 TB

**Medium Deployment (Small Production):**
- **Nodes**: 4
- **CPU per node**: 16 cores
- **RAM per node**: 64-128 GB
- **Storage per node**: 500 GB - 1 TB
- **Network**: 10 Gbps
- **Use case**: Small to medium production workloads
- **Data capacity**: 1-10 TB

**Large Deployment (Enterprise Production):**
- **Nodes**: 8-16
- **CPU per node**: 32 cores
- **RAM per node**: 256-512 GB
- **Storage per node**: 5-10 TB
- **Network**: 25-50 Gbps
- **Use case**: Large production workloads, data warehouses
- **Data capacity**: 10-100 TB

**Very Large Deployment (Enterprise Data Warehouse):**
- **Nodes**: 16-64
- **CPU per node**: 64 cores
- **RAM per node**: 512 GB - 1 TB
- **Storage per node**: 10-20 TB
- **Network**: 50-100 Gbps
- **Use case**: Very large data warehouses, big data analytics
- **Data capacity**: 100+ TB

**See also**: [Sizing Guidelines](https://docs.exasol.com/db/latest/administration/on-premise/sizing.htm)

---

## Performance Considerations

### Hardware Performance Factors

**CPU:**
- More cores = better parallel query execution
- Higher clock speed = faster single-threaded operations
- Exasol scales linearly with CPU cores
- Recommendation: Maximize core count within budget

**Memory:**
- Larger datasets fit in memory
- Reduces disk I/O for frequently accessed data
- Better performance for complex queries
- Recommendation: Size for dataset + 20-30% overhead

**Storage:**
- **SSD/NVMe dramatically faster** than HDD for data volumes
- Separate physical disks for data and archive improves performance
- RAID 0 (striping) can improve sequential read/write
- Recommendation: Use SSD/NVMe for all production deployments

**Network:**
- 10 Gbps minimum for production
- 25+ Gbps recommended for large clusters
- Low latency critical for cluster communication
- Recommendation: Dedicated network for cluster communication

### OS Configuration for Performance

**Disable Transparent Huge Pages:**
```bash
# Check current setting
cat /sys/kernel/mm/transparent_hugepage/enabled

# Disable (required for Exasol)
echo never | sudo tee /sys/kernel/mm/transparent_hugepage/enabled
echo never | sudo tee /sys/kernel/mm/transparent_hugepage/defrag

# Make persistent
cat > /etc/systemd/system/disable-thp.service << 'EOF'
[Unit]
Description=Disable Transparent Huge Pages (THP)
DefaultDependencies=no
After=sysinit.target local-fs.target
Before=basic.target

[Service]
Type=oneshot
ExecStart=/bin/sh -c 'echo never > /sys/kernel/mm/transparent_hugepage/enabled'
ExecStart=/bin/sh -c 'echo never > /sys/kernel/mm/transparent_hugepage/defrag'

[Install]
WantedBy=basic.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable disable-thp.service
```

**Configure I/O Scheduler for SSDs:**
```bash
# For SSD/NVMe, use none or noop scheduler
echo none | sudo tee /sys/block/nvme0n1/queue/scheduler

# Make persistent with udev rule
cat > /etc/udev/rules.d/60-scheduler.rules << 'EOF'
ACTION=="add|change", KERNEL=="nvme[0-9]*", ATTR{queue/scheduler}="none"
ACTION=="add|change", KERNEL=="sd[a-z]", ATTR{queue/rotational}=="0", ATTR{queue/scheduler}="none"
EOF

sudo udevadm control --reload-rules
```

---

## Cloud Platform Considerations

### AWS (Amazon Web Services)

**Instance Types:**
- **Compute optimized**: c5, c5n, c6i (CPU-intensive workloads)
- **Memory optimized**: r5, r6i, x1e (memory-intensive workloads)
- **Storage optimized**: i3, i4i (I/O-intensive workloads)

**Storage:**
- **EBS gp3**: General purpose SSD (good performance)
- **EBS io2**: Provisioned IOPS SSD (best performance)
- **Instance store**: NVMe SSD (highest performance, not persistent)

**Network:**
- Use **Enhanced Networking** (sr-iov)
- Enable **Placement Groups** for low latency

**See also**: [Deploy Exasol on AWS](https://docs.exasol.com/db/latest/administration/aws/installation.htm)

### Azure (Microsoft Azure)

**VM Sizes:**
- **Fsv2-series**: Compute optimized
- **Esv4-series**: Memory optimized
- **Lsv2-series**: Storage optimized

**Storage:**
- **Premium SSD (P30, P40, P50)**: Best performance
- **Ultra Disk**: Highest performance, configurable IOPS
- **Local NVMe**: Highest performance (not persistent)

**Network:**
- Use **Accelerated Networking**
- Deploy in same **Availability Zone** for low latency

### Google Cloud Platform (GCP)

**Machine Types:**
- **c2**: Compute optimized
- **m2**: Memory optimized
- **n2**: General purpose

**Storage:**
- **SSD Persistent Disks**: Good performance
- **Local SSD**: Highest performance (not persistent)

**Network:**
- Use **VM with high network bandwidth**
- Deploy in same **Zone** for low latency

---

## Troubleshooting Requirements Issues

### CPU/Architecture Issues

**Problem**: CPU doesn't support SSSE3
```bash
# Check CPU features
lscpu | grep -i ssse3
cat /proc/cpuinfo | grep -i ssse3

# If missing, cannot install Exasol
# Solution: Use different hardware or VM
```

### Memory Issues

**Problem**: Insufficient memory
```bash
# Check available memory
free -h
cat /proc/meminfo | grep MemTotal

# Solution: Add more RAM or reduce cluster size
```

### Storage Issues

**Problem**: Device names not persistent
```bash
# Check device paths
ls -l /dev/disk/by-id/

# Solution: Use LVM or /dev/disk/by-id/ paths in config
```

**Problem**: Insufficient disk space
```bash
# Check disk space
df -h

# Check partition sizes
lsblk

# Solution: Add more storage or clean up existing data
```

### Network Issues

**Problem**: Time not synchronized
```bash
# Check time on all nodes
for node in n0011 n0012 n0013 n0014; do
  echo "$node: $(ssh $node date)"
done

# Solution: Configure and enable NTP/chronyd
```

**Problem**: Ports blocked by firewall
```bash
# Check if port is open
nc -zv n0012 8563
telnet n0012 8563

# Solution: Configure firewall rules or disable firewall
```

### OS Configuration Issues

**Problem**: SELinux blocks operations
```bash
# Check SELinux status
getenforce
# If "Enforcing", set to Permissive

sudo setenforce 0
```

**Problem**: AppArmor parameter not disabled (Ubuntu 22.04/24.04)
```bash
# Check parameter
sysctl kernel.apparmor_restrict_unprivileged_userns
# Should be: 0

# Fix if needed
echo 'kernel.apparmor_restrict_unprivileged_userns = 0' | \
  sudo tee /etc/sysctl.d/91-exasol-apparmor-userns.conf
sudo sysctl -q --system
```

---

## Quick Reference

### Minimum Requirements Summary

```
CPU:        64-bit x86 with SSSE3, 4+ cores
RAM:        16 GB (64+ GB recommended)
Storage:    100 GB data + 100 GB archive
OS:         Ubuntu 20.04/22.04/24.04 or RHEL 8/9/10
Network:    1 Gbps (10+ Gbps recommended)
```

### Essential Commands

```bash
# Check CPU features
lscpu | grep -i ssse3

# Check memory
free -h

# Check disk space
df -h
lsblk

# Check network interfaces
ip addr show

# Check NTP status
timedatectl status

# Check SELinux (RHEL)
getenforce

# Check AppArmor parameter (Ubuntu 22.04/24.04)
sysctl kernel.apparmor_restrict_unprivileged_userns

# List persistent device paths
ls -l /dev/disk/by-id/

# Check OS version
cat /etc/os-release

# Test network connectivity
ping n0012
nc -zv n0012 8563
```

### Pre-Installation Validation Script

```bash
#!/bin/bash
# Exasol Pre-Installation Check Script

echo "=== Exasol Pre-Installation Validation ==="

# CPU check
echo -n "CPU SSSE3 support: "
if grep -q ssse3 /proc/cpuinfo; then
    echo "[OK]"
else
    echo "[FAIL] - SSSE3 not supported"
fi

# Memory check
echo -n "RAM (minimum 16 GB): "
mem_gb=$(free -g | awk '/^Mem:/{print $2}')
if [ $mem_gb -ge 16 ]; then
    echo "[OK] - ${mem_gb} GB"
else
    echo "[WARN] - Only ${mem_gb} GB available"
fi

# Disk space check
echo -n "OS disk space (minimum 150 GB): "
space_gb=$(df -BG / | awk 'NR==2 {print $4}' | sed 's/G//')
if [ $space_gb -ge 150 ]; then
    echo "[OK] - ${space_gb} GB free"
else
    echo "[WARN] - Only ${space_gb} GB free"
fi

# Network check
echo -n "Network interfaces (10 Gbps recommended): "
ip link show | grep -E "UP|state UP" | wc -l
echo " interfaces up"

# NTP check
echo -n "NTP synchronization: "
if timedatectl status | grep -q "synchronized: yes"; then
    echo "[OK]"
else
    echo "[WARN] - Time not synchronized"
fi

# SELinux check (if present)
if command -v getenforce &> /dev/null; then
    echo -n "SELinux: "
    status=$(getenforce)
    if [ "$status" == "Permissive" ] || [ "$status" == "Disabled" ]; then
        echo "[OK] - $status"
    else
        echo "[WARN] - $status (should be Permissive)"
    fi
fi

# OS version check
echo "OS: $(cat /etc/os-release | grep PRETTY_NAME | cut -d'"' -f2)"

echo "=== Validation Complete ==="
```

---

## Related Documentation

- [Planning Guide](https://docs.exasol.com/db/latest/planning.htm)
- [Install Exasol - Step by Step](https://docs.exasol.com/db/latest/administration/on-premise/installation/install_hw.htm)
- [Sizing Guidelines](https://docs.exasol.com/db/latest/administration/on-premise/sizing.htm)
- [Rootless Deployment](https://docs.exasol.com/db/latest/administration/on-premise/installation/install_rootless.htm)
- [GPU Support System Requirements](https://docs.exasol.com/db/latest/administration/on-premise/gpu_support/gpu_support_system_requirements.htm)
- [GPU Support Installation](https://docs.exasol.com/db/latest/administration/on-premise/gpu_support.htm)
- [Configure Network](https://docs.exasol.com/db/latest/administration/on-premise/installation/install_as_app/configure_network.htm)
- [Prepare Storage Devices](https://docs.exasol.com/db/latest/administration/on-premise/installation/install_as_app/configure_storage.htm)
- [Deploy Exasol on AWS](https://docs.exasol.com/db/latest/administration/aws/installation.htm)

## Common Questions

- What are the minimum system requirements for Exasol?
- What CPU is required for Exasol installation?
- How much RAM does Exasol need?
- What storage types are supported by Exasol?
- Can I install Exasol on NFS storage?
- What Linux distributions are supported for Exasol?
- Can I install Exasol on Ubuntu?
- Can I install Exasol on Red Hat Enterprise Linux?
- What network speed is required for Exasol cluster?
- What ports does Exasol use?
- How do I configure persistent device names for storage?
- Do I need to disable firewall for Exasol?
- What is the AppArmor requirement for Ubuntu?
- Can I install Exasol on virtual machines?
- Can I install Exasol on AWS, Azure, or Google Cloud?
- What are the requirements for a jump host?
- How much disk space does Exasol need?
- What is the difference between data and archive volumes?
- Can I use HDD for Exasol storage?
- Should I use SSD or NVMe for Exasol?
- How do I check if my CPU supports SSSE3?
- What is LVM and why is it recommended?
- How many nodes do I need for an Exasol cluster?
- Can I run Exasol on a single node?

## Summary

Exasol system requirements include:
- **CPU**: 64-bit x86 with SSSE3 support, 4+ cores (16+ recommended)
- **RAM**: 16 GB minimum, 64+ GB recommended for production
- **Storage**: 100+ GB per volume, SSD/NVMe recommended, persistent device names required
- **Network**: 1 Gbps minimum, 10+ Gbps recommended, static IPs, NTP required
- **OS**: Ubuntu 20.04/22.04/24.04 LTS or RHEL 8/9/10 with specific configuration
- **Configuration**: Firewall disabled/configured, SELinux permissive, antivirus disabled, proper umask

**Critical configurations:**
- NTP synchronized across all nodes
- Persistent device names for storage (LVM recommended)
- AppArmor parameter disabled (Ubuntu 22.04/24.04, Exasol < 8.34.0)
- Ports 22, 443, 8563, and 10000-10999 accessible

**Performance recommendations:**
- Use SSD or NVMe for production
- 10+ Gbps network for production clusters
- Identical hardware specifications across all nodes
- Separate physical disks for data and archive volumes

**For help**: [Create a support case](https://exasol.my.site.com/s/create-new-case)
