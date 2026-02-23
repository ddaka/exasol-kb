# Exasol Installation Procedure (On-Premise)

**Category:** Administration  
**Topic:** Installation, Deployment, Setup, Configuration, Infrastructure  
**Keywords:** install, deployment, c4, Exasol Deployment Tool, on-premise, hardware, VM, cloud, cluster, nodes, network, storage, SSH, configuration, setup  
**Source:** [Exasol Installation Documentation](https://docs.exasol.com/db/latest/administration/on-premise/installation/install_hw.htm)

## Overview

This document provides a comprehensive step-by-step guide for installing Exasol database in an **on-premise deployment** environment. Whether deploying on physical hardware or virtual machines in cloud services (AWS, Azure, Google Cloud), this guide covers the complete installation workflow from preparation through post-installation configuration.

**Deployment Scope:**
- Physical hardware installations
- Virtual machine deployments (AWS, Azure, Google Cloud)
- Single-node or multi-node cluster configurations
- Standard and rootless installations

**Key Components:**
- **c4 (Exasol Deployment Tool)**: Command-line tool for deploying and managing Exasol
- **ConfD**: Configuration daemon for cluster management
- **COS (Cluster Operating System)**: Underlying cluster management layer
- **Database Layer**: Exasol database software

**Installation Stages:**

| Stage | Description | Key Activities |
|-------|-------------|----------------|
| **Preparation** | Environment setup | Network configuration, user creation, SSH setup, storage preparation |
| **Installation** | Software deployment | Download c4, create configuration, deploy to hosts |
| **Post-Installation** | Initial configuration | License upload, backup setup, database connection |

**Important**: Read all prerequisites and planning guidelines before starting the installation.

---

## Installation Methods Overview

Exasol supports different installation approaches depending on your environment and security requirements:

| Installation Type | Description | Use Case | Documentation |
|-------------------|-------------|----------|---------------|
| **Standard (Root)** | Default installation with root privileges | Most common deployments | This document |
| **Rootless** | Installation for non-root users | Enhanced security environments | [Rootless Installation](https://docs.exasol.com/db/latest/administration/on-premise/installation/install_rootless.htm) |
| **GPU Support** | Installation with GPU acceleration for UDFs | Machine learning, data science workloads | [GPU Support](https://docs.exasol.com/db/latest/administration/on-premise/gpu_support.htm) |

This document focuses on **standard installation** using the c4 Deployment Tool.

---

## Prerequisites

### System Requirements

Before beginning installation, ensure all hosts meet the following minimum requirements:

#### Supported Linux Distributions

- **Red Hat Enterprise Linux (RHEL)** 8.x, 9.x
- **Ubuntu** 20.04 LTS, 22.04 LTS
- **SUSE Linux Enterprise Server (SLES)** 15 SP3+

See [System Requirements](https://docs.exasol.com/db/latest/administration/on-premise/installation/system_requirements.htm) for complete specifications.

#### Hardware Requirements

**Minimum Configuration (Testing/Development):**
- **CPU**: 4 cores per node
- **RAM**: 16 GB per node
- **Storage**: 100 GB available disk space per node
- **Network**: 1 Gbps network connectivity

**Recommended Configuration (Production):**
- **CPU**: 16+ cores per node
- **RAM**: 64+ GB per node
- **Storage**: 500+ GB SSD storage per node (separate data and archive volumes)
- **Network**: 10 Gbps network connectivity

**Multi-Node Cluster:**
- Minimum: 4 nodes
- Recommended: 4-64 nodes
- All nodes should have identical hardware specifications

#### Network Requirements

**Required Ports:**

| Port | Protocol | Purpose | Direction |
|------|----------|---------|-----------|
| **22** | TCP | SSH access | Inbound/Outbound |
| **443** | TCP | Exasol Admin UI (HTTPS) | Inbound |
| **8563** | TCP | Database connections | Inbound |
| **20000** | TCP | BucketFS HTTP | Inbound (optional) |
| **2580** | TCP | BucketFS HTTPS | Inbound (optional) |
| **10000-10999** | TCP | Internal cluster communication | Between nodes |

**Network Configuration:**
- **Private network**: Required for internal cluster communication between nodes
- **Public network**: Optional, for external client connections
- **DNS**: Hostname resolution for all nodes (recommended)
- **NTP**: Time synchronization across all nodes (required)

#### Storage Requirements

**Storage Devices:**
- **Data volumes**: Primary database storage (SSD recommended)
- **Archive volumes**: Backup storage (can be HDD)
- **System partition**: 50+ GB for OS and Exasol software

**Disk Space Allocation:**

| Purpose | Minimum | Recommended |
|---------|---------|-------------|
| **System/OS** | 50 GB | 100 GB |
| **Data volumes** | 100 GB per node | 500+ GB per node |
| **Archive volumes** | 100 GB per node | Equal to data volume size |
| **Installation workspace** | 20 GB | 50 GB |

**Important**: Separate physical disks for data and archive volumes recommended for performance.

#### User Requirements

**Installation User:**
- Standard installation: User with **sudo privileges** (root access)
- Rootless installation: Non-root user with specific capabilities

**SSH Access:**
- **Key-based authentication** configured (passwordless login recommended)
- SSH access from jump host to all database nodes OR
- SSH access between all database nodes (peer-to-peer)

### Planning Considerations

#### Architecture Planning

**Single-Node vs Multi-Node:**

```
Single-Node Architecture:
┌─────────────────────────────┐
│   Exasol Database Node      │
│  ┌──────────────────────┐   │
│  │  Database Instance   │   │
│  ├──────────────────────┤   │
│  │  Data Volume         │   │
│  │  Archive Volume      │   │
│  └──────────────────────┘   │
└─────────────────────────────┘

Multi-Node Cluster (4 nodes):
┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
│  Node 1  │  │  Node 2  │  │  Node 3  │  │  Node 4  │
│  Active  │  │  Active  │  │  Active  │  │  Active  │
│  n0011   │  │  n0012   │  │  n0013   │  │  n0014   │
└────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘
     └─────────────┴─────────────┴─────────────┘
            Private Network (10 Gbps)
```

**Recommended**: Multi-node cluster for high availability and horizontal scalability

#### Deployment Architecture

**Jump Host Deployment:**

```
┌─────────────────────────────────────────────────────┐
│                    Jump Host                        │
│              (Installation System)                  │
│  ┌──────────────────────────────────────────────┐   │
│  │   c4 Deployment Tool                         │   │
│  │   Configuration Files                        │   │
│  └──────────────────┬───────────────────────────┘   │
└─────────────────────┼───────────────────────────────┘
                      │ SSH
         ┌────────────┼────────────┐
         │            │            │
    ┌────▼───┐  ┌────▼───┐  ┌────▼───┐
    │ Node 1 │  │ Node 2 │  │ Node 3 │
    │        │  │        │  │        │
    └────────┘  └────────┘  └────────┘
```

**On-Node Deployment:**

```
┌──────────────────────────────────────────┐
│           Database Node 1                │
│  ┌────────────────────────────────────┐  │
│  │   c4 Deployment Tool               │  │
│  │   Configuration Files              │  │
│  └────────────┬───────────────────────┘  │
└───────────────┼──────────────────────────┘
                │ SSH
       ┌────────┼────────┐
       │        │        │
  ┌────▼───┐  ┌▼────────▼┐
  │ Node 2 │  │  Node 3  │
  │        │  │          │
  └────────┘  └──────────┘
```

#### Security Planning

**Firewall Configuration:**
- Open required ports between nodes
- Restrict external access to necessary ports only
- Use private IPs for cluster communication

**Authentication:**
- SSH key-based authentication (no passwords)
- Database user management post-installation
- TLS/SSL for client connections (recommended)

**See also**: [Planning Guide](https://docs.exasol.com/db/latest/planning.htm)

---

## Installation Workflow

### Overview

The complete installation process consists of **7 steps** across three stages:

```
┌─────────────────────────────────────────────────────────────┐
│                    PREPARATION STAGE                        │
├─────────────────────────────────────────────────────────────┤
│ Step 1: Configure network              [30-60 min]         │
│ Step 2: Create installation user       [15-30 min]         │
│ Step 3: Set up SSH authentication      [30-45 min]         │
│ Step 4: Prepare storage devices        [30-60 min]         │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                   INSTALLATION STAGE                        │
├─────────────────────────────────────────────────────────────┤
│ Step 5: Download and install c4        [15-30 min]         │
│ Step 6: Create configuration           [30-60 min]         │
│ Step 7: Deploy to hosts                [30-90 min]         │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                POST-INSTALLATION STAGE                      │
├─────────────────────────────────────────────────────────────┤
│ • Upload license                       [10 min]            │
│ • Configure backups                    [30 min]            │
│ • Connect to database                  [15 min]            │
│ • Additional admin tasks               [varies]            │
└─────────────────────────────────────────────────────────────┘

Total estimated time: 4-8 hours (depending on cluster size)
```

---

## Preparation Stage

### Step 1: Configure the Network

**Objective**: Configure network settings for inter-node communication and external access.

#### 1.1 Configure Hostname Resolution

**On each database node**, configure hostnames and IP addresses.

**Edit /etc/hosts:**
```bash
sudo vi /etc/hosts
```

**Add entries for all nodes:**
```
# Private network (cluster communication)
10.0.1.11  n0011  n0011.exasol.local
10.0.1.12  n0012  n0012.exasol.local
10.0.1.13  n0013  n0013.exasol.local
10.0.1.14  n0014  n0014.exasol.local

# Public network (client access) - optional
203.0.113.11  n0011-pub
203.0.113.12  n0012-pub
203.0.113.13  n0013-pub
203.0.113.14  n0014-pub
```

#### 1.2 Configure Network Interfaces

**Verify network interfaces:**
```bash
ip addr show
```

**Example configuration for private network interface:**
```bash
# RHEL/CentOS - /etc/sysconfig/network-scripts/ifcfg-eth1
DEVICE=eth1
TYPE=Ethernet
ONBOOT=yes
BOOTPROTO=static
IPADDR=10.0.1.11
NETMASK=255.255.255.0
```

**Restart networking:**
```bash
# RHEL/CentOS
sudo systemctl restart network

# Ubuntu
sudo systemctl restart networking
```

#### 1.3 Configure Firewall Rules

**RHEL/CentOS (firewalld):**
```bash
# Allow SSH
sudo firewall-cmd --permanent --add-service=ssh

# Allow database port
sudo firewall-cmd --permanent --add-port=8563/tcp

# Allow Exasol Admin UI
sudo firewall-cmd --permanent --add-port=443/tcp

# Allow internal cluster communication
sudo firewall-cmd --permanent --add-port=10000-10999/tcp

# Allow BucketFS (optional)
sudo firewall-cmd --permanent --add-port=2580/tcp
sudo firewall-cmd --permanent --add-port=20000/tcp

# Reload firewall
sudo firewall-cmd --reload
```

**Ubuntu (ufw):**
```bash
# Allow SSH
sudo ufw allow 22/tcp

# Allow database port
sudo ufw allow 8563/tcp

# Allow Exasol Admin UI
sudo ufw allow 443/tcp

# Allow internal cluster communication
sudo ufw allow 10000:10999/tcp

# Allow BucketFS (optional)
sudo ufw allow 2580/tcp
sudo ufw allow 20000/tcp

# Enable firewall
sudo ufw enable
```

#### 1.4 Verify Connectivity

**Test connectivity between nodes:**
```bash
# From each node, ping all other nodes
ping -c 3 n0012
ping -c 3 n0013
ping -c 3 n0014

# Verify DNS resolution
nslookup n0012
host n0013
```

**Verify time synchronization:**
```bash
# Check NTP status
timedatectl status

# Verify time sync is active
systemctl status chronyd  # RHEL/CentOS
systemctl status systemd-timesyncd  # Ubuntu
```

**See also**: [Configure Network](https://docs.exasol.com/db/latest/administration/on-premise/installation/install_as_app/configure_network.htm)

---

### Step 2: Create the Installation User

**Objective**: Create a dedicated user for Exasol installation and operation.

#### 2.1 Create User on All Nodes

**Execute on EACH database node:**

```bash
# Create Exasol user with sudo privileges
sudo useradd -m -s /bin/bash exasol

# Set password (optional, not needed if using SSH keys)
sudo passwd exasol

# Add user to sudo group
# RHEL/CentOS:
sudo usermod -aG wheel exasol

# Ubuntu:
sudo usermod -aG sudo exasol
```

#### 2.2 Configure Sudo Privileges

**Edit sudoers file for passwordless sudo:**
```bash
sudo visudo
```

**Add the following line:**
```
exasol ALL=(ALL) NOPASSWD: ALL
```

**Alternatively, create a dedicated sudoers file:**
```bash
sudo sh -c 'echo "exasol ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/exasol'
sudo chmod 440 /etc/sudoers.d/exasol
```

#### 2.3 Verify User Creation

**Test sudo access:**
```bash
# Switch to exasol user
sudo su - exasol

# Test sudo access (should not prompt for password)
sudo whoami
# Output: root

# Check user groups
groups
# Output: exasol wheel (or sudo)
```

**See also**: [Create Installation User](https://docs.exasol.com/db/latest/administration/on-premise/installation/install_as_app/create_installation_user.htm)

---

### Step 3: Set Up SSH Authentication

**Objective**: Configure passwordless SSH access between jump host and all nodes, or between all nodes.

#### 3.1 Generate SSH Key Pair

**On the jump host or primary installation node:**

```bash
# Switch to exasol user
sudo su - exasol

# Generate SSH key pair (press Enter to accept defaults)
ssh-keygen -t rsa -b 4096 -C "exasol@installation"

# Output:
# Generating public/private rsa key pair.
# Enter file in which to save the key (/home/exasol/.ssh/id_rsa): [Enter]
# Enter passphrase (empty for no passphrase): [Enter]
# Enter same passphrase again: [Enter]
```

**Verify key creation:**
```bash
ls -la ~/.ssh/
# id_rsa (private key)
# id_rsa.pub (public key)
```

#### 3.2 Copy Public Key to All Nodes

**Method 1: Using ssh-copy-id (recommended):**
```bash
# Copy to each node
ssh-copy-id exasol@n0011
ssh-copy-id exasol@n0012
ssh-copy-id exasol@n0013
ssh-copy-id exasol@n0014
```

**Method 2: Manual copy:**
```bash
# Display public key
cat ~/.ssh/id_rsa.pub

# On each target node, create .ssh directory and append key
ssh exasol@n0011
mkdir -p ~/.ssh
chmod 700 ~/.ssh
# Paste the public key into ~/.ssh/authorized_keys
echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDexample..." >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
exit
```

#### 3.3 Configure SSH Settings

**Edit SSH config for convenience (optional):**
```bash
cat > ~/.ssh/config << 'EOF'
Host n00*
    User exasol
    StrictHostKeyChecking no
    UserKnownHostsFile /dev/null
    IdentityFile ~/.ssh/id_rsa
EOF

chmod 600 ~/.ssh/config
```

#### 3.4 Verify SSH Access

**Test passwordless SSH to all nodes:**
```bash
# Should connect without password prompt
ssh n0011 hostname
ssh n0012 hostname
ssh n0013 hostname
ssh n0014 hostname

# Test sudo access via SSH
ssh n0011 sudo whoami
# Output: root
```

**Important**: Installation will fail if SSH authentication is not properly configured.

**See also**: [Configure SSH Authentication](https://docs.exasol.com/db/latest/administration/on-premise/installation/install_as_app/configure_ssh.htm)

---

### Step 4: Prepare Storage Devices

**Objective**: Identify and prepare storage devices for data and archive volumes.

#### 4.1 Identify Storage Devices

**On each database node, list available block devices:**
```bash
# List all block devices
lsblk

# Example output:
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0   100G  0 disk 
├─sda1   8:1    0    99G  0 part /
└─sda2   8:2    0     1G  0 part [SWAP]
sdb      8:16   0   500G  0 disk 
sdc      8:32   0   500G  0 disk 

# Show detailed disk information
sudo fdisk -l

# Check for existing partitions
sudo parted -l
```

**Identify disks for Exasol:**
- **sdb**: Data volume (e.g., /dev/sdb)
- **sdc**: Archive volume (e.g., /dev/sdc)

#### 4.2 Prepare Storage Devices

**CRITICAL**: The disks should be **raw block devices** without partitions or filesystems.

**If disks have existing partitions, remove them:**

**WARNING**: This will destroy all data on the disk!

```bash
# Remove existing partitions
sudo wipefs -a /dev/sdb
sudo wipefs -a /dev/sdc

# Or use parted
sudo parted /dev/sdb mklabel gpt
sudo parted /dev/sdc mklabel gpt
```

**Verify disks are clean:**
```bash
# Should show no partitions
sudo parted /dev/sdb print
sudo parted /dev/sdc print
```

#### 4.3 Set Disk Permissions

**Ensure Exasol user has access to storage devices:**
```bash
# Check current permissions
ls -l /dev/sd*

# Add Exasol user to disk group
sudo usermod -aG disk exasol

# Alternatively, set specific permissions (not recommended)
# sudo chown exasol:exasol /dev/sdb /dev/sdc
```

#### 4.4 Configure Disk Scheduler (Optional, Recommended for Performance)

**For SSD storage, use noop or none scheduler:**
```bash
# Check current scheduler
cat /sys/block/sdb/queue/scheduler

# Set to none (for SSDs)
echo none | sudo tee /sys/block/sdb/queue/scheduler
echo none | sudo tee /sys/block/sdc/queue/scheduler

# Make persistent across reboots
cat > /etc/udev/rules.d/60-scheduler.rules << 'EOF'
ACTION=="add|change", KERNEL=="sd[b-z]", ATTR{queue/scheduler}="none"
EOF
```

**See also**: [Prepare Storage Devices](https://docs.exasol.com/db/latest/administration/on-premise/installation/install_as_app/configure_storage.htm)

---

## Installation Stage

### Step 5: Download and Install c4

**Objective**: Download and install the c4 Deployment Tool.

#### 5.1 Download c4 Binary

**Choose installation location:**
- **Jump host**: Install c4 on external system
- **Database node**: Install c4 on one of the database nodes

**Switch to Exasol user:**
```bash
sudo su - exasol
```

**Download c4:**
```bash
# Create installation directory
mkdir -p ~/exasol-install
cd ~/exasol-install

# Download latest c4 version
wget https://downloads.exasol.com/c4/latest/c4-linux-amd64
# Or for specific version:
# wget https://downloads.exasol.com/c4/0.5.0/c4-linux-amd64

# Make executable
chmod +x c4-linux-amd64

# Create symlink for convenience
ln -s c4-linux-amd64 c4
```

#### 5.2 Verify c4 Installation

**Check c4 version:**
```bash
./c4 version

# Example output:
# > c4 version: 0.5.0
# > Build date: 2025-01-15
# > Git commit: a1b2c3d4
```

**Display help:**
```bash
./c4 --help
./c4 play --help
```

#### 5.3 Configure c4 Path (Optional)

**Add c4 to PATH for convenience:**
```bash
# Add to ~/.bashrc
echo 'export PATH=$HOME/exasol-install:$PATH' >> ~/.bashrc
source ~/.bashrc

# Verify
c4 version
```

**See also**: [Download and Install c4](https://docs.exasol.com/db/latest/administration/on-premise/installation/install_as_app/download_install_c4.htm), [Install c4](https://docs.exasol.com/db/latest/administration/on-premise/c4/c4_install.htm)

---

### Step 6: Create Configuration File

**Objective**: Create a configuration file defining the cluster topology and settings.

#### 6.1 Generate Base Configuration

**c4 can generate a template configuration:**
```bash
./c4 play create-config -t hardware -o config.yaml
```

#### 6.2 Edit Configuration File

**Complete configuration example for 4-node cluster:**

```yaml
# Exasol Cluster Configuration
version: 1

# Cluster identification
cluster:
  name: exasol-production
  license: ""  # Will upload after installation

# Database version to deploy
image: "@exasol-8.34.0"

# Node definitions
nodes:
  - name: n0011
    private_ip: 10.0.1.11
    public_ip: 203.0.113.11  # Optional, for client access
    ssh:
      host: n0011
      user: exasol
      private_key_file: ~/.ssh/id_rsa
    disks:
      - name: data
        devices: ["/dev/sdb"]
      - name: archive
        devices: ["/dev/sdc"]
        
  - name: n0012
    private_ip: 10.0.1.12
    public_ip: 203.0.113.12
    ssh:
      host: n0012
      user: exasol
      private_key_file: ~/.ssh/id_rsa
    disks:
      - name: data
        devices: ["/dev/sdb"]
      - name: archive
        devices: ["/dev/sdc"]
        
  - name: n0013
    private_ip: 10.0.1.13
    public_ip: 203.0.113.13
    ssh:
      host: n0013
      user: exasol
      private_key_file: ~/.ssh/id_rsa
    disks:
      - name: data
        devices: ["/dev/sdb"]
      - name: archive
        devices: ["/dev/sdc"]
        
  - name: n0014
    private_ip: 10.0.1.14
    public_ip: 203.0.113.14
    ssh:
      host: n0014
      user: exasol
      private_key_file: ~/.ssh/id_rsa
    disks:
      - name: data
        devices: ["/dev/sdb"]
      - name: archive
        devices: ["/dev/sdc"]

# Network configuration
network:
  # Private network for cluster communication
  private:
    subnet: 10.0.1.0/24
    gateway: 10.0.1.1
  
  # Public network for client connections (optional)
  public:
    subnet: 203.0.113.0/24
    gateway: 203.0.113.1

# Database configuration
databases:
  - name: PROD_DB
    version: "8.34.0"
    dataVolume:
      size: "100GB"
      disk: data
      redundancy: 2  # Number of replicas
    
    archiveVolume:
      size: "100GB"
      disk: archive
      
    # Memory allocation
    memorySize: "32GB"
    
    # Port configuration
    port: 8563
    
# BucketFS configuration (optional)
bucketfs:
  - name: bfsdefault
    http_port: 20000
    https_port: 2580
    buckets:
      - name: default
        public: false
        read_password: "readpw"
        write_password: "writepw"

# Exasol Admin UI configuration (optional)
admin_ui:
  enabled: true
  port: 443
  tls:
    enabled: true
    cert_file: ""  # Path to certificate (optional)
    key_file: ""   # Path to private key (optional)

# Optional: GPU support
gpu:
  enabled: false
```

#### 6.3 Single-Node Configuration Example

**For single-node deployments:**

```yaml
version: 1

cluster:
  name: exasol-dev

image: "@exasol-8.34.0"

nodes:
  - name: n0011
    private_ip: 10.0.1.11
    ssh:
      host: localhost
      user: exasol
    disks:
      - name: data
        devices: ["/dev/sdb"]
      - name: archive
        devices: ["/dev/sdc"]

databases:
  - name: DEV_DB
    dataVolume:
      size: "50GB"
      disk: data
      redundancy: 1
    archiveVolume:
      size: "50GB"
      disk: archive
    memorySize: "16GB"
    port: 8563
```

#### 6.4 Validate Configuration

**Validate syntax and settings:**
```bash
# Check YAML syntax
./c4 play check-config -f config.yaml

# Perform dry run (validates without deploying)
./c4 play -f config.yaml --dry-run
```

**See also**: [Create Configuration](https://docs.exasol.com/db/latest/administration/on-premise/installation/install_as_app/create_configuration.htm)

---

### Step 7: Deploy to Hosts

**Objective**: Deploy Exasol to all nodes and start the cluster.

#### 7.1 Perform Dry Run (Recommended)

**Validate configuration without making changes:**
```bash
./c4 play -f config.yaml --dry-run

# Review output for any warnings or errors
```

**Expected output:**
```
[INFO] Validating configuration...
[INFO] Checking SSH connectivity to all nodes...
[OK] SSH connection to n0011 successful
[OK] SSH connection to n0012 successful
[OK] SSH connection to n0013 successful
[OK] SSH connection to n0014 successful
[INFO] Verifying storage devices...
[OK] /dev/sdb available on n0011
[OK] /dev/sdc available on n0011
[OK] /dev/sdb available on n0012
...
[INFO] Dry run completed successfully
[INFO] Configuration is valid and ready for deployment
```

#### 7.2 Execute Deployment

**CRITICAL**: Ensure you have reviewed and validated the configuration.

**Start deployment:**
```bash
./c4 play -f config.yaml

# Alternative: Specify log output
./c4 play -f config.yaml --log-file install.log
```

**Deployment progress:**
```
[INFO] Starting Exasol deployment...
[INFO] Downloading Exasol image: @exasol-8.34.0
[====================================] 100% (2.5 GB)
[INFO] Deploying to node n0011...
[INFO] Installing Exasol software...
[====================================] 100%
[INFO] Deploying to node n0012...
[====================================] 100%
[INFO] Deploying to node n0013...
[====================================] 100%
[INFO] Deploying to node n0014...
[====================================] 100%
[INFO] Configuring cluster network...
[INFO] Initializing storage devices...
[INFO] Creating database volumes...
[INFO] Starting Exasol services...
[INFO] Starting database PROD_DB...
[SUCCESS] Deployment completed successfully!
[INFO] Play ID: c4d5e6f7-8901-2345-6789-0abcdef12345
```

**Estimated deployment time:**
- **4-node cluster**: 30-90 minutes
- **Single node**: 15-30 minutes

#### 7.3 Monitor Deployment

**In another terminal, monitor logs:**
```bash
# Watch deployment logs
tail -f install.log

# Check cluster status during deployment
./c4 ps
```

#### 7.4 Verify Deployment

**After deployment completes, verify cluster status:**

```bash
# Check cluster status
./c4 ps

# Example output:
  N  PLAY_ID   NODE  MEDIUM  INSTANCE     DB_VERSION  EXTERNAL_IP     INTERNAL_IP  STAGE  STATE    UPTIME    TTL
┌─  1  c4d5e6f7  11    hwcf    bare-metal   8.34.0      203.0.113.11    10.0.1.11    d      running  00:05:23  +∞
│   1  c4d5e6f7  12    hwcf    bare-metal   8.34.0      203.0.113.12    10.0.1.12    d      running  00:05:24  +∞
│   1  c4d5e6f7  13    hwcf    bare-metal   8.34.0      203.0.113.13    10.0.1.13    d      running  00:05:24  +∞
└─  1  c4d5e6f7  14    hwcf    bare-metal   8.34.0      203.0.113.14    10.0.1.14    d      running  00:05:24  +∞

# Check database status
./c4 connect -i c4d5e6f7 -s cos -- 'confd_client db_state db_name: PROD_DB'

# Expected output:
# running
```

**Save Play ID for future operations:**
```bash
# Save Play ID to file
echo "c4d5e6f7" > play_id.txt
```

**See also**: [Deploy to Hosts](https://docs.exasol.com/db/latest/administration/on-premise/installation/install_as_app/deploy_on_hosts.htm)

---

## Post-Installation Stage

### Upload License

**Objective**: Upload a valid Exasol license to enable full functionality.

#### Default License Limitations

**Trial license included with installation:**
- **Data limit**: 10 GiB of raw data
- **Time limit**: Typically 30 days
- **Feature access**: Full feature set

#### Upload License File

**Obtain license from Exasol:**
- Contact Exasol sales or support
- License file format: XML

**Method 1: Using Exasol Admin UI**

1. Access Admin UI: `https://<node-ip>` or `https://<hostname>`
2. Log in with admin credentials
3. Navigate to **Configuration** → **License**
4. Click **Upload License**
5. Select license XML file
6. Click **Apply**

**Method 2: Using confd_client (Command Line)**

```bash
# Copy license file to node
scp license.xml exasol@n0011:~/

# Upload license
./c4 connect -i PLAY_ID -s cos -- 'confd_client db_add_license db_name: PROD_DB license_file: /home/exasol/license.xml'
```

**Verify license:**
```bash
# Check license details
./c4 connect -i PLAY_ID -s cos -- 'confd_client db_info db_name: PROD_DB'

# Or via SQL
# SELECT * FROM EXA_SYSTEM_EVENTS WHERE EVENT_TYPE = 'LICENSE';
```

**See also**: [Upload License](https://docs.exasol.com/db/latest/administration/on-premise/installation/install_as_app/upload_license.htm)

---

### Configure Backups

**Objective**: Set up automated backup schedule for data protection.

#### Backup Strategy

**Backup types:**
- **Database backup**: Full database export
- **Remote backup**: Backup to external storage (recommended)
- **BucketFS backup**: Backup of UDF scripts and files

#### Create Backup Schedule

**Using confd_client:**

```bash
# Add remote backup schedule (recommended)
./c4 connect -i PLAY_ID -s cos -- 'confd_client db_backup_add_schedule \
  db_name: PROD_DB \
  backup_name: daily_backup \
  backup_type: remote \
  volume_name: DataVolume1 \
  backup_dir: "s3://bucket-name/exasol-backups/" \
  level: 0 \
  enabled: true \
  minute: 0 \
  hour: 2 \
  day: * \
  month: * \
  weekday: *'

# Create weekly full backup
./c4 connect -i PLAY_ID -s cos -- 'confd_client db_backup_add_schedule \
  db_name: PROD_DB \
  backup_name: weekly_full \
  backup_type: remote \
  volume_name: DataVolume1 \
  backup_dir: "s3://bucket-name/exasol-backups/" \
  level: 0 \
  enabled: true \
  minute: 0 \
  hour: 1 \
  day: * \
  month: * \
  weekday: 0'
```

**Schedule explanation:**
- **minute**: 0-59 (* = every minute)
- **hour**: 0-23 (hour of day)
- **day**: 1-31 (day of month)
- **month**: 1-12 (* = every month)
- **weekday**: 0-6 (0 = Sunday)

**Example schedules:**
- Daily at 2 AM: `0 2 * * *`
- Weekly on Sunday at 1 AM: `0 1 * * 0`
- Every 6 hours: `0 */6 * * *`

#### Verify Backup Schedule

```bash
# List all backup schedules
./c4 connect -i PLAY_ID -s cos -- 'confd_client db_backup_list db_name: PROD_DB'

# Run manual backup test
./c4 connect -i PLAY_ID -s cos -- 'confd_client db_backup db_name: PROD_DB volume_name: DataVolume1 backup_dir: "s3://bucket-name/test/"'
```

**See also**: [Configure Backups](https://docs.exasol.com/db/latest/administration/on-premise/installation/install_as_app/configure_backups.htm), [Backup and Restore](https://docs.exasol.com/db/latest/administration/on-premise/backup_restore.htm)

---

### Connect to Database

**Objective**: Establish initial connection and verify database functionality.

#### Connection Information

**Database connection details:**
- **Host**: Any node's public IP or hostname (e.g., `203.0.113.11`)
- **Port**: `8563` (default)
- **Database**: `PROD_DB` (or your configured database name)
- **Default user**: `sys`
- **Default password**: `exasol` (must be changed immediately)

#### Initial Connection Steps

**1. Change Default Password (CRITICAL)**

**Connect as sys user and change password:**
```sql
-- Using any SQL client, connect to:
-- Host: 203.0.113.11:8563
-- User: sys
-- Password: exasol

-- Change sys password immediately
ALTER USER sys IDENTIFIED BY 'NewSecurePassword123!';
```

**2. Create Database Users**

```sql
-- Create admin user
CREATE USER admin_user IDENTIFIED BY 'SecurePassword456!';
GRANT DBA TO admin_user;

-- Create application user
CREATE USER app_user IDENTIFIED BY 'AppPassword789!';
GRANT CREATE SESSION TO app_user;

-- Create read-only user
CREATE USER readonly_user IDENTIFIED BY 'ReadPassword012!';
GRANT CREATE SESSION TO readonly_user;
```

**3. Create Initial Schema**

```sql
-- Connect as admin_user

-- Create application schema
CREATE SCHEMA app_schema;

-- Grant permissions
GRANT ALL ON SCHEMA app_schema TO app_user;
GRANT SELECT ON SCHEMA app_schema TO readonly_user;

-- Create sample table
CREATE TABLE app_schema.test_table (
    id INTEGER,
    name VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Insert test data
INSERT INTO app_schema.test_table (id, name) VALUES (1, 'Test Record');

-- Verify
SELECT * FROM app_schema.test_table;
```

#### Connection Methods

**JDBC Connection:**
```
jdbc:exa:203.0.113.11:8563
```

**ODBC Connection String:**
```
DRIVER=Exasol;EXAHOST=203.0.113.11:8563;UID=app_user;PWD=AppPassword789!;
```

**Python (using pyexasol):**
```python
import pyexasol

conn = pyexasol.connect(
    dsn='203.0.113.11:8563',
    user='app_user',
    password='AppPassword789!',
    schema='app_schema'
)

# Test query
result = conn.execute('SELECT * FROM test_table')
print(result.fetchall())

conn.close()
```

**See also**: [Connect to Exasol](https://docs.exasol.com/db/latest/administration/on-premise/installation/install_as_app/connect_exasol.htm)

---

### Additional Administrative Tasks

**Post-installation administration checklist:**

#### User Management

```sql
-- Create roles
CREATE ROLE data_analyst;
GRANT SELECT ON SCHEMA app_schema TO data_analyst;

-- Assign roles to users
GRANT data_analyst TO readonly_user;

-- Review user permissions
SELECT * FROM EXA_DBA_USERS;
SELECT * FROM EXA_DBA_ROLE_PRIVS;
```

#### Volume Management

```bash
# List volumes
./c4 connect -i PLAY_ID -s cos -- 'confd_client storage_list_volumes'

# Add additional data volume (if needed)
./c4 connect -i PLAY_ID -s cos -- 'confd_client storage_add_volume \
  volume_name: DataVolume2 \
  volume_type: data \
  size: 200GB \
  disk: data \
  nodes: n0011,n0012,n0013,n0014'
```

#### Monitoring Setup

```sql
-- Enable auditing
ALTER SYSTEM SET AUDITING = TRUE;

-- Configure system monitoring
SELECT * FROM EXA_MONITOR_LAST_DAY;
SELECT * FROM EXA_SYSTEM_EVENTS;

-- Check database statistics
SELECT * FROM EXA_DB_SIZE_LAST_DAY;
```

#### Performance Tuning

```bash
# Check resource usage
./c4 connect -i PLAY_ID -s cos -- 'confd_client db_statistics db_name: PROD_DB'

# Review configuration
./c4 connect -i PLAY_ID -s cos -- 'confd_client db_info db_name: PROD_DB'
```

**See also**: [Administration Overview](https://docs.exasol.com/db/latest/administration/on-premise/administration.htm)

---

## Troubleshooting

### Common Installation Issues

#### Issue: SSH Connection Failed

**Error message:**
```
[ERROR] Failed to connect to node n0012 via SSH
[ERROR] Permission denied (publickey)
```

**Causes:**
- SSH key not copied to target node
- SSH key permissions incorrect
- SSH daemon not running
- Firewall blocking port 22

**Solutions:**
```bash
# Verify SSH key exists
ls -la ~/.ssh/id_rsa*

# Check key permissions (must be 600 for private key)
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub

# Test SSH connection manually
ssh -v exasol@n0012

# Copy SSH key again
ssh-copy-id exasol@n0012

# Verify authorized_keys on target node
ssh n0012 'cat ~/.ssh/authorized_keys'
chmod 600 ~/.ssh/authorized_keys
```

#### Issue: Storage Device Not Found

**Error message:**
```
[ERROR] Device /dev/sdb not found on node n0011
```

**Causes:**
- Disk not attached to system
- Incorrect device path in configuration
- Device already mounted
- Insufficient permissions

**Solutions:**
```bash
# List all block devices
lsblk
sudo fdisk -l

# Check if device is mounted
mount | grep sdb

# Unmount if necessary
sudo umount /dev/sdb

# Verify device path
ls -l /dev/sd*

# Update configuration with correct device path
vi config.yaml
```

#### Issue: Insufficient Disk Space

**Error message:**
```
[ERROR] Insufficient disk space for installation
[ERROR] Required: 100 GB, Available: 45 GB
```

**Causes:**
- Not enough free space on system partition
- Logs consuming space
- Previous failed installation artifacts

**Solutions:**
```bash
# Check disk space on all partitions
df -h

# Clean up package cache
sudo apt clean  # Ubuntu
sudo yum clean all  # RHEL/CentOS

# Remove old logs
sudo journalctl --vacuum-time=7d

# Clean up temporary files
sudo rm -rf /tmp/*
sudo rm -rf /var/tmp/*

# Remove old Exasol installations (if any)
rm -rf ~/.ccc/*
```

#### Issue: Port Already in Use

**Error message:**
```
[ERROR] Port 8563 is already in use
```

**Causes:**
- Previous Exasol installation still running
- Another service using the port
- Firewall rule conflict

**Solutions:**
```bash
# Check what's using the port
sudo netstat -tulpn | grep 8563
sudo lsof -i :8563

# Stop conflicting service
sudo systemctl stop <service-name>

# Or choose different port in configuration
vi config.yaml
# Change: port: 8564
```

#### Issue: Database Won't Start

**Error message:**
```
[ERROR] Database PROD_DB failed to start
[ERROR] State: offline
```

**Causes:**
- Insufficient memory
- Storage volume errors
- Configuration errors
- License issues

**Solutions:**
```bash
# Check database state
./c4 connect -i PLAY_ID -s cos -- 'confd_client db_state db_name: PROD_DB'

# Review logs
./c4 connect -t1/host
tail -100 /exa/logs/cored/*/bucketd.log
tail -100 /exa/logs/db/PROD_DB/bucketd.log

# Check volume status
./c4 connect -i PLAY_ID -s cos -- 'confd_client storage_list_volumes'

# Try manual start
./c4 connect -i PLAY_ID -s cos -- 'confd_client db_start db_name: PROD_DB'

# If memory issue, reduce allocation
./c4 connect -i PLAY_ID -s cos -- 'confd_client db_configure \
  db_name: PROD_DB \
  params: "mem_size=24GB"'
```

#### Issue: Deployment Hangs

**Symptoms:**
- Deployment process stops responding
- No progress for extended period
- Terminal shows no output

**Solutions:**
```bash
# Check deployment process
ps aux | grep c4

# Check network connectivity
ping n0011
ping n0012

# Check system resources
ssh n0011 'top -bn1 | head -20'
ssh n0011 'df -h'

# Review logs in another terminal
tail -f install.log

# If truly hung, may need to cancel and restart
# Press Ctrl+C
# Clean up partial installation
./c4 rm -P -f config.yaml
# Restart deployment
./c4 play -f config.yaml
```

### Getting Help

**Collect diagnostic information:**
```bash
# Gather system information
./c4 connect -i PLAY_ID -s cos -- 'confd_client db_info db_name: PROD_DB' > db_info.txt
./c4 ps > cluster_status.txt

# Collect logs
./c4 connect -t1/host
tar -czf exasol-logs.tar.gz /exa/logs/

# Export configuration
cp config.yaml config-backup.yaml
```

**Create support case:**
- Visit: [Exasol Support Portal](https://exasol.my.site.com/s/create-new-case)
- Include: Logs, configuration, error messages, system details

---

## Best Practices

### Pre-Installation

**Planning:**
- Document cluster architecture and network topology
- Create detailed implementation plan with rollback procedures
- Review system requirements thoroughly
- Plan for future growth (storage, compute, nodes)

**Testing:**
- Test installation in staging environment first
- Verify all prerequisites before starting
- Validate network connectivity between all nodes
- Test storage device performance

**Security:**
- Use dedicated VLANs for cluster communication
- Implement firewall rules before installation
- Use strong passwords for all users
- Configure TLS/SSL for client connections
- Enable database auditing

**Backup Planning:**
- Plan backup strategy before deployment
- Test backup and restore procedures
- Use remote backups for critical data
- Document backup retention policies

### During Installation

**Best practices:**
- Use jump host for installation when possible
- Always perform dry run before actual deployment
- Monitor installation logs in real-time
- Don't interrupt deployment process
- Use `screen` or `tmux` for long-running operations

**Configuration:**
- Use descriptive names for cluster and databases
- Follow naming conventions consistently
- Document all configuration parameters
- Keep backup of configuration file
- Use version control for configuration files

### Post-Installation

**Immediate tasks:**
- Change all default passwords
- Upload production license
- Configure backup schedules
- Set up monitoring and alerting
- Create initial database schema
- Document connection details

**Ongoing maintenance:**
- Monitor cluster health regularly
- Review logs for warnings/errors
- Keep Exasol version up to date
- Test backup restoration periodically
- Review and audit user access
- Monitor disk space usage
- Plan capacity upgrades proactively

**Performance:**
- Allocate appropriate memory to databases
- Use separate physical disks for data and archive
- Configure appropriate redundancy levels
- Monitor query performance
- Optimize table distribution
- Use statistics and indexes appropriately

---

## Quick Reference

### Essential Commands

```bash
# Installation
./c4 play -f config.yaml                    # Deploy cluster
./c4 play -f config.yaml --dry-run          # Validate config
./c4 ps                                     # Show cluster status

# Database operations
./c4 connect -i PLAY_ID -s cos -- 'confd_client db_start db_name: PROD_DB'
./c4 connect -i PLAY_ID -s cos -- 'confd_client db_stop db_name: PROD_DB'
./c4 connect -i PLAY_ID -s cos -- 'confd_client db_state db_name: PROD_DB'
./c4 connect -i PLAY_ID -s cos -- 'confd_client db_info db_name: PROD_DB'

# Storage operations
./c4 connect -i PLAY_ID -s cos -- 'confd_client storage_list_volumes'
./c4 connect -i PLAY_ID -s cos -- 'confd_client storage_check_volumes'

# Backup operations
./c4 connect -i PLAY_ID -s cos -- 'confd_client db_backup_list db_name: PROD_DB'
./c4 connect -i PLAY_ID -s cos -- 'confd_client db_backup db_name: PROD_DB volume_name: DataVolume1 backup_dir: "s3://bucket/"'

# System information
./c4 connect -t1/host                       # SSH to first node
./c4 version                                # Show c4 version
```

### Connection Strings

**JDBC:**
```
jdbc:exa:<host>:8563;schema=<schema>
```

**ODBC:**
```
DRIVER=Exasol;EXAHOST=<host>:8563;UID=<user>;PWD=<password>;
```

**Python:**
```python
import pyexasol
conn = pyexasol.connect(dsn='<host>:8563', user='<user>', password='<password>')
```

### Configuration Template (Minimal)

```yaml
version: 1
cluster:
  name: my-cluster
image: "@exasol-8.34.0"
nodes:
  - name: n0011
    private_ip: 10.0.1.11
    ssh:
      host: n0011
      user: exasol
    disks:
      - name: data
        devices: ["/dev/sdb"]
      - name: archive
        devices: ["/dev/sdc"]
databases:
  - name: MY_DB
    dataVolume:
      size: "100GB"
      disk: data
    archiveVolume:
      size: "100GB"
      disk: archive
    memorySize: "32GB"
    port: 8563
```

### Verification Checklist

**After installation:**
- [ ] All nodes show "running" state in `c4 ps`
- [ ] Database is running: `confd_client db_state`
- [ ] Can connect via SQL client
- [ ] Default passwords changed
- [ ] License uploaded (if required)
- [ ] Backup schedule configured
- [ ] Monitoring set up
- [ ] Documentation updated

---

## Installation Decision Tree

```
What type of installation do you need?
│
├─ Single-node (testing/development)
│  ├─ Hardware: 1 server, 2+ disks
│  ├─ Config: 1 node, single database
│  └─ Time: ~30 minutes
│
├─ Multi-node cluster (production)
│  ├─ Hardware: 4+ servers, 2+ disks per server
│  ├─ Config: Multiple nodes, redundancy
│  └─ Time: ~2-4 hours
│
├─ Rootless installation (enhanced security)
│  ├─ Requires: Non-root user configuration
│  └─ See: Rootless Installation guide
│
└─ GPU-enabled (ML/AI workloads)
   ├─ Requires: GPU hardware, CUDA drivers
   └─ See: GPU Support guide

Installation location?
│
├─ From jump host (recommended)
│  ├─ Install c4 on external system
│  ├─ Configure SSH to all nodes
│  └─ Specify config file with -i option
│
└─ From database node
   ├─ Install c4 on one of the nodes
   ├─ Configure SSH to other nodes
   └─ Run deployment from that node
```

---

## Related Documentation

- [System Requirements](https://docs.exasol.com/db/latest/administration/on-premise/installation/system_requirements.htm)
- [Planning Guide](https://docs.exasol.com/db/latest/planning.htm)
- [Install c4](https://docs.exasol.com/db/latest/administration/on-premise/c4/c4_install.htm)
- [Using c4](https://docs.exasol.com/db/latest/administration/on-premise/c4/using_c4.htm)
- [Rootless Installation](https://docs.exasol.com/db/latest/administration/on-premise/installation/install_rootless.htm)
- [GPU Support](https://docs.exasol.com/db/latest/administration/on-premise/gpu_support.htm)
- [Configure Network](https://docs.exasol.com/db/latest/administration/on-premise/installation/install_as_app/configure_network.htm)
- [Create Installation User](https://docs.exasol.com/db/latest/administration/on-premise/installation/install_as_app/create_installation_user.htm)
- [Configure SSH](https://docs.exasol.com/db/latest/administration/on-premise/installation/install_as_app/configure_ssh.htm)
- [Prepare Storage](https://docs.exasol.com/db/latest/administration/on-premise/installation/install_as_app/configure_storage.htm)
- [Download c4](https://docs.exasol.com/db/latest/administration/on-premise/installation/install_as_app/download_install_c4.htm)
- [Create Configuration](https://docs.exasol.com/db/latest/administration/on-premise/installation/install_as_app/create_configuration.htm)
- [Deploy to Hosts](https://docs.exasol.com/db/latest/administration/on-premise/installation/install_as_app/deploy_on_hosts.htm)
- [Upload License](https://docs.exasol.com/db/latest/administration/on-premise/installation/install_as_app/upload_license.htm)
- [Configure Backups](https://docs.exasol.com/db/latest/administration/on-premise/installation/install_as_app/configure_backups.htm)
- [Connect to Exasol](https://docs.exasol.com/db/latest/administration/on-premise/installation/install_as_app/connect_exasol.htm)
- [Administration Overview](https://docs.exasol.com/db/latest/administration/on-premise/administration.htm)
- [Backup and Restore](https://docs.exasol.com/db/latest/administration/on-premise/backup_restore.htm)
- [Update Procedure](https://docs.exasol.com/db/latest/administration/on-premise/updates.htm)

## Common Questions

- How do I install Exasol on-premise?
- What are the prerequisites for installing Exasol?
- How much disk space is required for Exasol installation?
- What Linux distributions are supported for Exasol?
- How do I configure network settings for Exasol cluster?
- What is the c4 deployment tool?
- How do I create an Exasol configuration file?
- How do I deploy Exasol to multiple nodes?
- How long does Exasol installation take?
- What is the difference between single-node and multi-node installation?
- How do I set up SSH authentication for Exasol installation?
- What storage devices are required for Exasol?
- How do I prepare disks for Exasol installation?
- Can I install Exasol on cloud VMs (AWS, Azure, Google Cloud)?
- How do I perform a dry run before deployment?
- What is a rootless Exasol installation?
- How do I install Exasol with GPU support?
- How do I configure BucketFS during installation?
- What ports need to be open for Exasol?
- How do I verify Exasol installation was successful?
- How do I upload a license after installation?
- How do I configure backups after installing Exasol?
- How do I connect to Exasol database after installation?
- What should I do if installation fails?
- How do I troubleshoot SSH connection errors during installation?

## Summary

Installing Exasol on-premise involves:
- **Preparation**: Configure network, create users, set up SSH, prepare storage (2-3 hours)
- **Installation**: Download c4, create configuration, deploy to hosts (1-2 hours)
- **Post-Installation**: Upload license, configure backups, connect to database (1 hour)

**Installation stages overview:**
1. **Prerequisites**: Verify hardware, Linux distribution, network, storage
2. **Preparation**: Network → Users → SSH → Storage
3. **Installation**: c4 tool → Configuration → Deployment
4. **Post-Installation**: License → Backups → Connection → Administration

**Key tools:**
- **c4**: Exasol Deployment Tool for installation and management
- **confd_client**: Configuration and cluster management
- **Exasol Admin UI**: Web-based administration interface

**Critical requirements:**
- Supported Linux distribution (RHEL, Ubuntu, SLES)
- Adequate hardware (CPU, RAM, storage)
- Network connectivity between nodes
- SSH key-based authentication
- Raw block devices for storage

**Installation time:**
- Single-node: ~1-2 hours
- Multi-node cluster: ~4-8 hours

**For help**: [Create a support case](https://exasol.my.site.com/s/create-new-case)
