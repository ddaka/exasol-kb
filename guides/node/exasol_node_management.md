# Exasol Node Management - Complete Guide

**Category:** Administration  
**Topic:** Node Management, Cluster Operations, High Availability  
**Keywords:** nodes, add nodes, remove nodes, replace nodes, reserve nodes, active nodes, data nodes, suspend nodes, resume nodes, failover, hot standby, node states, cluster management, c4, confd_client  
**Source:** Exasol On-Premise Administration Documentation

## Overview

This comprehensive guide covers all aspects of managing nodes in Exasol on-premise deployments, including adding, removing, replacing, suspending, and maintaining nodes in a cluster.

**Node Types:**
- **Active (Data) Nodes**: Nodes that actively participate in database operations and store data
- **Reserve Nodes**: Standby nodes that automatically take over if an active node fails (hot standby)

**Management Tools:**
- **c4**: Exasol Deployment Tool for infrastructure operations
- **confd_client**: Command-line tool for cluster configuration and management
- **systemctl**: System service management

---

## Table of Contents

1. [Node Concepts and Architecture](#node-concepts-and-architecture)
2. [Node States](#node-states)
3. [Prerequisites and Setup](#prerequisites-and-setup)
4. [Stop and Start Nodes](#stop-and-start-nodes)
5. [Add and Activate Data Nodes](#add-and-activate-data-nodes)
6. [Add Reserve Nodes](#add-reserve-nodes)
7. [Replace Failed Nodes](#replace-failed-nodes)
8. [Suspend and Resume Nodes](#suspend-and-resume-nodes)
9. [Node Verification and Monitoring](#node-verification-and-monitoring)
10. [Troubleshooting Node Issues](#troubleshooting-node-issues)
11. [Best Practices](#best-practices)

---

## Node Concepts and Architecture

### Active vs Reserve Nodes

**Active (Data) Nodes:**
- Participate in database operations
- Store database data
- Process queries
- Can be part of running databases

**Reserve Nodes:**
- Stand by for active nodes (hot standby)
- Automatically take over if an active node fails
- Do not store data or process queries while in standby
- Provide fail safety for the cluster

### Fail Safety Architecture

Exasol uses **hot standby** failover:
1. Reserve nodes monitor active nodes
2. If an active node fails, a reserve node immediately takes over
3. Cluster operating system (COS) restarts services on the new node
4. Minimal downtime during failover

**Redundancy levels:**
- `k=1`: Can tolerate 1 node failure
- `k=2`: Can tolerate 2 node failures
- Higher k values require more reserve nodes

For detailed information about fail safety, see [Fail safety documentation](https://docs.exasol.com/db/latest/planning/fail_safety.htm).

### Deployment Stages

When monitoring nodes with `c4 ps`, you'll see stage indicators:

| Stage | Description | Node State |
|-------|-------------|------------|
| **i** | Image deployed | Initial deployment |
| **c** | COS running | Cluster OS active, database not running |
| **d** | Database running | Fully operational |

---

## Node States

### Common Node States

| State | Description | Typical Cause |
|-------|-------------|---------------|
| **online** | Node is active and operational | Normal operation |
| **offline** | Node is not reachable | Network issue, hardware failure, intentional shutdown |
| **suspended** | Node intentionally taken offline | Maintenance, manual suspension |
| **running** | Node services are running | Normal operation (synonym for online) |

### Checking Node States

**Via ConfD:**
```bash
# List all nodes
confd_client node_list

# Get detailed node information
confd_client node_info nid: 11
```

**Via c4:**
```bash
# Check deployment status
c4 ps

# Example output:
#   N  PLAY_ID   NODE  MEDIUM  INSTANCE     DB_VERSION  EXTERNAL_IP     INTERNAL_IP  STAGE  STATE    UPTIME    TTL
# ┌─ 1  c3275f84  11    host    -            2025.1.0    203.0.113.11    10.0.0.11    d      running  03:50:12  +∞
# │  1  c3275f84  12    host    -            2025.1.0    203.0.113.12    10.0.0.12    d      running  03:50:13  +∞
# │  1  c3275f84  13    host    -            2025.1.0    203.0.113.13    10.0.0.13    d      running  03:50:13  +∞
# └─ 1  c3275f84  14    host    -            2025.1.0    203.0.113.14    10.0.0.14    d      running  03:50:13  +∞
```

**Via system tables (SQL):**
```sql
-- Check node states
SELECT * FROM EXA_SYSTEM_EVENTS WHERE MEASURE_TIME > CURRENT_TIMESTAMP - INTERVAL '1' HOUR;

-- Check cluster nodes
SELECT * FROM EXA_LOADAVG;
```

---

## Prerequisites and Setup

### General Prerequisites

**User requirements:**
- Root access or user with sudo privileges
- SSH access to cluster nodes
- Access to c4 configuration files

**Network requirements:**
- All nodes must be in the same subnet
- Consecutive, evenly spaced static IPv4 addresses
- Proper DNS/hostname resolution

**Configuration requirements:**
- Know the play ID of your deployment (`c4 ps`)
- Have the original c4 configuration file used for deployment
- Understand current cluster topology

### Connecting to COS

**All node management operations require connection to the Cluster Operating System (COS).**

**Find the play ID:**
```bash
c4 ps
```

**Connect to COS:**
```bash
c4 connect -i <PLAY_ID> -s cos

# Example:
./c4 connect -i c3275f84 -s cos
```

**Alternative: Direct SSH to node:**
```bash
ssh root@<NODE_IP> -p 20002

# Example:
ssh root@10.70.0.171 -p 20002
```

### Using Configuration Files

When working with existing deployments, you must use the **same configuration file** that was used during initial deployment.

**Set configuration path:**
```bash
# If config is not in current directory
export CCC_CONFIG=/path/to/config

# Or prepend to each command
CCC_CONFIG=/path/to/config c4 <command>

# Example:
CCC_CONFIG=./configs/production c4 ps
```

---

## Stop and Start Nodes

### Overview

You may need to stop nodes for:
- Hardware maintenance
- OS updates
- Network reconfiguration
- Power management

**Critical**: Always stop the database before stopping nodes to prevent data corruption.

### Prerequisites

- Root or sudo access
- Database must be stopped before stopping nodes

### Stop a Node

**Step 1: Stop the database**

```bash
# Connect to COS
c4 connect -i <PLAY_ID> -s cos

# Stop the database
confd_client db_stop db_name: <DB_NAME>

# Example:
confd_client db_stop db_name: MY_DATABASE
```

**Why stop the database first?**  
Stopping nodes while the database is running can leave it in a corrupted state, leading to data loss.

**Step 2: Stop Exasol services**

```bash
# Stop c4_cloud_command (Exasol software stack)
sudo systemctl stop c4_cloud_command

# Stop c4 service
sudo systemctl stop c4
```

**Note**: If you are root, omit `sudo`.

**Step 3: Verify services stopped**

```bash
# Check c4 status
systemctl status c4

# Check c4_cloud_command status
systemctl status c4_cloud_command
```

**Expected output when stopped:**
```
● c4.service - c4 server
     Loaded: loaded (/etc/systemd/system/c4.service; enabled; vendor preset: enabled)
     Active: inactive (dead) since Wed 2024-05-29 18:45:57 UTC; 29s ago
    Process: 26651 ExecStart=/var/lib/ccc/etc/c4 (code=killed, signal=TERM)
   Main PID: 26651 (code=killed, signal=TERM)
```

**Step 4: Shut down the host**

Once services are stopped, you can safely shut down the host system:
```bash
shutdown -h now
```

### Start a Node

**Step 1: Start the host system**

Power on the host. Exasol services should automatically start.

**Step 2: Verify services started**

```bash
# Check c4 status
systemctl status c4

# Check c4_cloud_command status
systemctl status c4_cloud_command
```

**Expected output when running:**
```
● c4.service - c4 server
     Loaded: loaded (/etc/systemd/system/c4.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2024-05-29 18:48:51 UTC; 2min 47s ago
```

**If services don't auto-start:**
```bash
# Start c4
sudo systemctl start c4

# Start c4_cloud_command
sudo systemctl start c4_cloud_command

# Verify
systemctl status c4
systemctl status c4_cloud_command
```

**Step 3: Check database status**

```bash
# Check deployment status
c4 ps

# Look for nodes in stage 'd' (database running)
```

**Step 4: Start database if needed**

If the database is not running (nodes not in stage 'd'):
```bash
# Connect to COS
c4 connect -i <PLAY_ID> -s cos

# Start the database
confd_client db_start db_name: <DB_NAME>

# Example:
confd_client db_start db_name: MY_DATABASE
```

### Out-of-Band Management

**Note**: Exasol does not include functionality to send out-of-band data to restart a host when the OS is unreachable.

For remote power management, set up IPMI services such as:
- **iDRAC** (Dell)
- **iLO** (HPE)
- **ILOM** (Oracle)

Refer to your hardware vendor's documentation for IPMI setup.

---

## Add and Activate Data Nodes

### Overview

Adding data nodes expands cluster capacity by:
- Adding more storage
- Increasing processing power
- Scaling the cluster horizontally

**Process summary:**
1. Prepare physical/virtual hosts
2. Reserve IP addresses
3. Add nodes to deployment (as inactive)
4. Add nodes as reserve nodes
5. Append nodes to data volume
6. Activate nodes as data nodes
7. Reorganize database

### Prerequisites

**Critical**: `CCC_HOST_CLEANUP` must be set to `false` in the deployment configuration.

**System requirements:**
- Prepare physical or virtual hosts (see [System Requirements](https://docs.exasol.com/db/latest/administration/on-premise/installation/system_requirements.htm))
- Consecutive, evenly spaced static IPv4 addresses
- Same subnet as existing nodes
- Same hardware specifications as existing nodes (recommended)

**Configuration:**
- Original c4 configuration file
- Access to c4 deployment tool
- SSH access to cluster

### Procedure

#### Step 1: Prepare the Hosts

**Physical hosts:**
- Install OS (if not pre-installed)
- Configure network settings
- Assign static IP addresses
- Ensure SSH access

**Virtual hosts:**
- Create VMs with appropriate resources
- Configure network
- Assign static IPs

**IP addressing:**
- Use consecutive IPs (e.g., if existing nodes are 10.0.0.11-13, new nodes should be 10.0.0.14-15)
- Same subnet as existing nodes
- Static (not DHCP)

#### Step 2: Reserve IP Addresses

Reserve private (and optionally public) IP addresses for new nodes.

**Find the play ID:**
```bash
CCC_CONFIG=config ./c4 ps

# Example output:
#   N  PLAY_ID   NODE  MEDIUM  INSTANCE     EXTERNAL_IP     INTERNAL_IP  STAGE  STATE      UPTIME    TTL
# ┌─ 1  3a4a7d8d  11    host    c5d.2xlarge  203.0.113.11    10.0.0.11    d      running    04:35:16  +∞
# │  1  3a4a7d8d  12    host    c5d.2xlarge  203.0.113.12    10.0.0.12    d      running    04:35:16  +∞
# └─ 1  3a4a7d8d  13    host    c5d.2xlarge  203.0.113.13    10.0.0.13    d      running    04:35:15  +∞
```

**Reserve addresses:**
```bash
CCC_CONFIG=config ./c4 host reserve \
  --ccc-host-reserved-addrs 10.0.0.14 \
  --ccc-host-reserved-addrs 10.0.0.15 \
  --ccc-host-reserved-external-addrs 203.0.113.14 \
  --ccc-host-reserved-external-addrs 203.0.113.15 \
  <PLAY_ID>

# Example:
CCC_CONFIG=config ./c4 host reserve \
  --ccc-host-reserved-addrs 10.0.0.14 \
  --ccc-host-reserved-addrs 10.0.0.15 \
  --ccc-host-reserved-external-addrs 203.0.113.14 \
  --ccc-host-reserved-external-addrs 203.0.113.15 \
  3a4a7d8d
```

**Parameters:**

| Parameter | Description | Required |
|-----------|-------------|----------|
| `--ccc-host-reserved-addrs` | Private IP address of new node (repeat for each node) | Yes |
| `--ccc-host-reserved-external-addrs` | Public IP address of new node (repeat for each node) | Only if existing nodes have public IPs |

**Output:**
```
INFO[2024-06-18 08:53:46] Reserving new nodes for deployment: ****************************
INFO[2024-06-18 08:53:49] Done
```

**Important notes:**
- Parameters MUST be defined on command line (not in config file or environment variables)
- Repeat `--ccc-host-reserved-addrs` for each new node
- If existing nodes have external IPs, you must also provide `--ccc-host-reserved-external-addrs`
- Values in config file will be overwritten by this command

#### Step 3: Connect to COS

```bash
CCC_CONFIG=config ./c4 connect -i <PLAY_ID> -s cos

# Example:
CCC_CONFIG=config ./c4 connect -i 3a4a7d8d -s cos
```

#### Step 4: Add Nodes to Deployment

Add nodes by cloning the configuration of an existing active node.

```bash
confd_client infra_instances_add nid: <EXISTING_NODE_ID> num_nodes: <NUM_NEW_NODES>

# Example: Clone node 11 to create 2 new nodes
confd_client infra_instances_add nid: 11 num_nodes: 2
```

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `nid` | integer | ID of an existing active node to clone |
| `num_nodes` | integer | Number of new nodes to create |

**What happens:**
- New nodes are created with configuration from the specified node
- Nodes are numbered consecutively after the highest existing node ID
- Nodes automatically start and reach stage **c** (COS running, database not running)
- Nodes are NOT yet part of the cluster

**Verification:**
```bash
c4 ps

# New nodes should appear with stage 'c'
```

#### Step 5: Add Nodes as Reserve Nodes

Add the new nodes to the cluster as reserve nodes first.

```bash
confd_client db_add_reserve_nodes db_name: <DB_NAME> node_list: '[<NODE_ID_1>, <NODE_ID_2>]'

# Example: Add nodes 14 and 15 as reserve nodes
confd_client db_add_reserve_nodes db_name: MY_DATABASE node_list: '[14, 15]'
```

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `db_name` | string | Name of the database |
| `node_list` | list | List of node IDs (integers) to add as reserve nodes |

**Note**: Use single quotes around the list to prevent shell interpretation.

#### Step 6: Append Nodes to Data Volume

Append the new nodes to the existing data volume's address space.

**First, get the volume name:**
```bash
confd_client st_volume_list | grep name

# Output example:
# name: DataVolume1
```

**Then append nodes:**
```bash
confd_client st_volume_append_node vname: <VOLUME_NAME> node_num: <NUM_NODES> node_ids: '[<NODE_ID_1>, <NODE_ID_2>]'

# Example:
confd_client st_volume_append_node vname: DataVolume1 node_num: 2 node_ids: '[14, 15]'
```

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `vname` | string | Name of the data volume |
| `node_num` | integer | Number of nodes being added |
| `node_ids` | list | List of node IDs (integers) |

#### Step 7: Stop the Database

**Critical**: The database must be stopped before activating nodes as data nodes.

```bash
confd_client db_stop db_name: <DB_NAME>

# Example:
confd_client db_stop db_name: MY_DATABASE
```

**Verification:**
```bash
c4 ps

# Nodes should change from stage 'd' to stage 'c'
```

#### Step 8: Activate Nodes as Data Nodes

Convert reserve nodes to active data nodes using `db_enlarge`.

```bash
confd_client db_enlarge db_name: <DB_NAME> num_new_nodes: <NUM_NODES>

# Example: Activate 2 nodes
confd_client db_enlarge db_name: MY_DATABASE num_new_nodes: 2
```

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `db_name` | string | Name of the database |
| `num_new_nodes` | integer | Number of nodes to activate as data nodes |

**What happens:**
- Reserve nodes are promoted to active data nodes
- Nodes become part of the database's active node pool
- Database configuration is updated

#### Step 9: Start the Database

```bash
confd_client db_start db_name: <DB_NAME>

# Example:
confd_client db_start db_name: MY_DATABASE
```

**Verification:**
```bash
c4 ps

# All nodes should be in stage 'd'
```

#### Step 10: Reorganize the Database

**Critical**: After adding data nodes, you MUST reorganize the database to redistribute data.

**Via SQL:**
```sql
REORGANIZE DATABASE;
```

**Via c4 SQL client:**
```bash
CCC_CONFIG=config ./c4 connect -t <PLAY_ID>/db

# In the SQL client:
> REORGANIZE DATABASE;
```

**What REORGANIZE does:**
- Redistributes data across all nodes (including new ones)
- Reconstitutes distribution and partitioning
- Balances load across the cluster

**Time estimate:**
- Depends on database size
- Can take minutes to hours for large databases
- Monitor progress via `EXA_STATISTICS.EXA_SYSTEM_EVENTS`

### Verification

**Check node list:**
```bash
confd_client node_list
```

**Check database configuration:**
```bash
confd_client db_info db_name: MY_DATABASE
```

**Check via SQL:**
```sql
-- Check cluster size
SELECT * FROM EXA_LOADAVG;

-- Should show all nodes including new ones
```

**Check c4 deployment:**
```bash
c4 ps

# All nodes should be in stage 'd' with state 'running'
```

---

## Add Reserve Nodes

### Overview

Reserve nodes provide fail safety through hot standby. If an active node fails, a reserve node automatically takes over.

**Two methods:**
1. **Define reserve nodes during initial deployment** (preferred)
2. **Add reserve nodes to existing deployment** (covered here)

### When to Add Reserve Nodes

Add reserve nodes to provide fail safety if:
- You initially deployed without reserve nodes
- You want to increase redundancy level (e.g., from k=1 to k=2)
- You need higher availability guarantees

### Procedure to Add Reserve Nodes to Existing Deployment

**This follows a similar process to adding data nodes, but stops after Step 5.**

**Summary:**
1. Prepare hosts
2. Reserve IP addresses (same as data nodes)
3. Connect to COS
4. Add nodes to deployment using `infra_instances_add`
5. Add nodes as reserve nodes using `db_add_reserve_nodes`
6. Append nodes to data volume using `st_volume_append_node`

**DO NOT:**
- Run `db_enlarge` (this activates nodes as data nodes)
- Stop the database (not required for reserve nodes)
- Run REORGANIZE (not needed for reserve nodes)

**Commands:**
```bash
# Step 1-2: Prepare hosts and reserve IPs (same as data nodes)
CCC_CONFIG=config ./c4 host reserve \
  --ccc-host-reserved-addrs 10.0.0.16 \
  <PLAY_ID>

# Step 3: Connect to COS
CCC_CONFIG=config ./c4 connect -i <PLAY_ID> -s cos

# Step 4: Add nodes to deployment
confd_client infra_instances_add nid: 11 num_nodes: 1

# Step 5: Add as reserve nodes
confd_client db_add_reserve_nodes db_name: MY_DATABASE node_list: '[16]'

# Step 6: Append to volume
confd_client st_volume_append_node vname: DataVolume1 node_num: 1 node_ids: '[16]'

# Verification
confd_client node_list
```

**Result**: Node 16 is now a reserve node, ready to take over if an active node fails.

### Define Reserve Nodes During Deployment (Preferred Method)

When creating a new deployment with c4, define reserve nodes in the configuration file:

**In config file:**
```bash
# Total number of nodes (active + reserve)
CCC_HOST_ADDRS="10.0.0.11 10.0.0.12 10.0.0.13 10.0.0.14"

# Number of active nodes (rest become reserve nodes)
# Example: 4 total nodes, 3 active = 1 reserve node
CCC_NUM_ACTIVE_NODES=3
```

**Result**: Nodes 11-13 are active, node 14 is reserve.

For detailed deployment instructions, see [Add reserve nodes at installation](https://docs.exasol.com/db/latest/administration/on-premise/nodes/reserve_nodes/add_reserve_nodes_installation.htm).

---

## Replace Failed Nodes

### Overview

Replace failed nodes with new hardware when:
- Hardware failure (disk, memory, CPU)
- Unrecoverable OS corruption
- Physical damage to server

**Limitations:**
- Bulk replacement of nodes is NOT supported (unexpected errors)
- Node renaming is NOT supported
- Replace one node at a time

### Prerequisites

- Database must be stopped
- New host with same IP addresses as failed node
- Original c4 configuration file
- Know which Exasol package version is currently installed

### Procedure

#### Step 1: Connect to COS

```bash
./c4 connect -i <PLAY_ID> -s cos

# Example:
./c4 connect -i c3275f84 -s cos
```

**Find play ID:**
```bash
c4 ps
```

#### Step 2: Stop the Database

**Critical**: Database must be stopped before replacing nodes.

```bash
confd_client db_stop db_name: <DB_NAME>

# Example:
confd_client db_stop db_name: MY_DATABASE
```

#### Step 3: Prepare the New Node

**Hardware setup:**
- Install new physical/virtual host
- Assign the EXACT SAME IP addresses as the failed node
  - Same private IP
  - Same public IP (if applicable)
- Ensure network connectivity
- Configure SSH access

**Why same IPs?**  
The cluster configuration references nodes by IP address. Using the same IPs allows seamless replacement without cluster reconfiguration.

#### Step 4: Update the Configuration

**Modify the c4 configuration file** used for the original deployment.

**Add the IP address of the node being replaced:**
```bash
# Existing configuration
CCC_HOST_ADDRS="10.10.10.11 10.10.10.12 10.10.10.13"

# Add the NEW parameter with the IP of node to replace
CCC_HOST_SETUP_ADDRS="10.10.10.12"

# Other required parameters
CCC_HOST_IMAGE_USER=exasol
CCC_HOST_IMAGE_PASSWORD=exasol123
CCC_HOST_KEY_PAIR_FILE=id_rsa
CCC_HOST_DATADISK=/dev/nvme2n1,/dev/nvme3n1
CCC_PLAY_WORKING_COPY=@exasol-8.32.0
```

**Important:**
- `CCC_HOST_SETUP_ADDRS` specifies which nodes to replace
- Value must exist in `CCC_HOST_ADDRS` or `CCC_HOST_EXTERNAL_ADDRS`
- `CCC_PLAY_WORKING_COPY` must match the currently installed Exasol version

**Example: Replace only node 10.10.10.12**

Original config:
```bash
CCC_HOST_ADDRS="10.10.10.11 10.10.10.12 10.10.10.13"
```

Updated config:
```bash
CCC_HOST_ADDRS="10.10.10.11 10.10.10.12 10.10.10.13"
CCC_HOST_SETUP_ADDRS="10.10.10.12"
CCC_HOST_IMAGE_USER=exasol
CCC_HOST_IMAGE_PASSWORD=exasol123
CCC_HOST_KEY_PAIR_FILE=id_rsa
CCC_HOST_DATADISK=/dev/nvme2n1,/dev/nvme3n1
CCC_PLAY_WORKING_COPY=@exasol-8.32.0
```

#### Step 5: Deploy the Replacement Node

Run c4 with the updated configuration to deploy the replacement node.

```bash
./c4 host play -i <CONFIG_FILE>

# Example with config file in current directory:
./c4 host play -i config

# Example with config file in another location:
./c4 host play -i /path_to_config_file/myconfig
```

**What happens:**
- c4 connects to the new node via SSH
- Installs Exasol software
- Configures the node with the same settings as before
- Starts Exasol services
- Node joins the cluster

**Monitor deployment:**
```bash
c4 ps

# Watch for node to reach stage 'c' or 'd'
```

#### Step 6: Start the Database

Once the node is deployed and services are running, start the database.

```bash
confd_client db_start db_name: <DB_NAME>

# Example:
confd_client db_start db_name: MY_DATABASE
```

**Verification:**
```bash
c4 ps

# Node should reach stage 'd' (database running)
```

### Verification

**Check node status:**
```bash
confd_client node_list

# Replaced node should show as 'online'
```

**Check database status:**
```bash
confd_client db_info db_name: MY_DATABASE
```

**Check via SQL:**
```sql
-- Check node is active
SELECT * FROM EXA_LOADAVG;

-- Check for errors
SELECT * FROM EXA_SYSTEM_EVENTS WHERE MEASURE_TIME > CURRENT_TIMESTAMP - INTERVAL '1' HOUR;
```

### Troubleshooting Node Replacement

**Node doesn't start:**
- Check network connectivity: `ping <NEW_NODE_IP>`
- Check SSH access: `ssh exasol@<NEW_NODE_IP>`
- Check c4 logs: Look for error messages during deployment

**Node starts but database doesn't:**
- Check EXAConf: `grep -A 50 "^\[Node :" /exa/etc/EXAConf`
- Check cored logs: `tail -100 /exa/logs/cored/exainit.log`
- Check ConfD logs: `tail -100 /exa/logs/logd/ConfD.log`

**Version mismatch:**
- Ensure `CCC_PLAY_WORKING_COPY` matches current version
- Check current version: `confd_client version_list`

---

## Suspend and Resume Nodes

### Overview

**Suspend** a node to intentionally take it offline for maintenance without stopping Exasol services.

**Use cases:**
- Planned maintenance
- Testing failover behavior
- Temporarily removing a node from the cluster

**Difference from stopping:**
- **Suspend**: Node marked as intentionally offline (state = suspended), Exasol services may continue running
- **Stop**: Node powered off, all services stopped

### Prerequisites

None. Suspending nodes is a lightweight operation.

### Suspend a Node

**Connect to COS:**
```bash
c4 connect -i <PLAY_ID> -s cos

# Example:
./c4 connect -i c3275f84 -s cos
```

**Suspend the node:**
```bash
confd_client node_suspend nid: <NODE_ID>

# Example: Suspend node 12
confd_client node_suspend nid: 12
```

**Verification:**
```bash
confd_client node_list

# Node 12 should show state 'suspended'
```

### Resume a Node

**Option 1: Manual resume**

```bash
confd_client node_resume nid: <NODE_ID>

# Example: Resume node 12
confd_client node_resume nid: 12
```

**Option 2: Automatic resume on startup**

Starting up a suspended node automatically unsuspends it.

**Verification:**
```bash
confd_client node_list

# Node 12 should show state 'online'
```

### Important Notes

**Suspended vs Offline:**
- **Suspended**: Intentional, controlled state
- **Offline**: Unexpected, could indicate a problem

**Reserve nodes:**
- Reserve nodes often show as 'suspended' when not in use (normal behavior)
- Any node (not just specific nodes like n14) can be a reserve node

**Monitoring:**
- Suspended nodes do NOT trigger alerts (intentional state)
- Offline nodes DO trigger alerts (unexpected state)

---

## Node Verification and Monitoring

### Checking Node Status

**Via ConfD:**
```bash
# List all nodes
confd_client node_list

# Get detailed node info
confd_client node_info nid: 11

# Check node state
confd_client node_state nid: 11
```

**Via c4:**
```bash
# Check deployment status
c4 ps

# Shows: node IDs, IPs, stage, state, uptime
```

**Via SQL:**
```sql
-- Check node load and status
SELECT * FROM EXA_LOADAVG;

-- Check system events
SELECT * FROM EXA_SYSTEM_EVENTS 
WHERE MEASURE_TIME > CURRENT_TIMESTAMP - INTERVAL '1' HOUR
ORDER BY MEASURE_TIME DESC;

-- Check database size per node
SELECT * FROM EXA_DB_SIZE_LAST_DAY;
```

**Via monitoring (if available):**
- Check Grafana dashboards
- Query InfluxDB metrics
- Check Loki logs

### Monitoring Node Health

**System-level checks:**
```bash
# Check CPU usage
top

# Check memory
free -h

# Check disk space
df -h

# Check network
ifconfig
netstat -tulpn
```

**Exasol-specific checks:**
```bash
# Check Exasol services
systemctl status c4
systemctl status c4_cloud_command

# Check logs
tail -f /exa/logs/logd/Health.log
tail -f /exa/logs/logd/eventd.log
tail -f /exa/logs/cored/exainit.log
```

**Database-level checks:**
```sql
-- Check for errors
SELECT * FROM EXA_SYSTEM_EVENTS 
WHERE EVENT_TYPE = 'ERROR'
  AND MEASURE_TIME > CURRENT_TIMESTAMP - INTERVAL '24' HOUR;

-- Check node states
SELECT * FROM EXA_MONITOR_LAST_DAY;

-- Check database connectivity
SELECT SESSION_ID, USER_NAME, CLIENT, STATUS 
FROM EXA_ALL_SESSIONS;
```

### Common Monitoring Metrics

**Node metrics:**
- CPU usage (per core and overall)
- Memory usage (used, free, cached)
- Disk I/O (read/write throughput)
- Network traffic (bytes in/out)
- Node state (online, offline, suspended)

**Database metrics:**
- Database state (running, stopped)
- Active sessions
- Query throughput
- Transaction rate
- Data volume usage

**Storage metrics:**
- Volume state (healthy, degraded)
- Disk state (healthy, failed)
- Free space
- I/O errors

### Alerting on Node Issues

**Key alerts to configure:**
- Node goes offline (unexpected)
- Node has high CPU/memory usage
- Node has disk space issues
- Node has network connectivity issues
- Database fails to start on node

**Example alert conditions:**
```sql
-- Node offline
SELECT * FROM EXA_SYSTEM_EVENTS 
WHERE EVENT_TYPE = 'ERROR'
  AND DBMS_TEXT LIKE '%node%offline%';

-- High CPU
SELECT * FROM EXA_MONITOR_LAST_DAY
WHERE CPU > 90;

-- Disk full
SELECT * FROM EXA_DB_SIZE_LAST_DAY
WHERE USE > 90;
```

---

## Troubleshooting Node Issues

### Node Won't Start

**Symptom**: Node doesn't reach stage 'd' after startup

**Checks:**
```bash
# Check c4 services
systemctl status c4
systemctl status c4_cloud_command

# Check initialization log
tail -100 /exa/logs/cored/exainit.log

# Check ConfD log
tail -100 /exa/logs/logd/ConfD.log | grep -i error

# Check network connectivity
ping <OTHER_NODE_IP>
```

**Common causes:**
- Network misconfiguration
- Services not starting
- EXAConf corruption
- Disk issues

**Solutions:**
1. **Start services manually:**
   ```bash
   systemctl start c4
   systemctl start c4_cloud_command
   ```

2. **Check EXAConf:**
   ```bash
   cat /exa/etc/EXAConf | grep -A 20 "^\[Node :"
   ```

3. **Check disk space:**
   ```bash
   df -h
   ```

### Database Won't Start on Node

**Symptom**: Node reaches stage 'c' but not 'd'

**Checks:**
```bash
# Check database status
confd_client db_info db_name: MY_DATABASE

# Check controller log
tail -100 /exa/logs/db/Exasol/*controller*

# Start database manually
confd_client db_start db_name: MY_DATABASE
```

**Common causes:**
- Database intentionally stopped
- License issues
- Volume issues
- Memory issues

### Node Shows as Offline

**Symptom**: Node shows 'offline' in `node_list`

**Checks:**
```bash
# Check node state
confd_client node_state nid: <NODE_ID>

# Check if node is reachable
ping <NODE_IP>
ssh root@<NODE_IP> -p 20002

# Check on the node itself
systemctl status c4
systemctl status c4_cloud_command
```

**Common causes:**
- Network issues
- Services crashed
- Hardware failure

**Solutions:**
1. **Restart services:**
   ```bash
   ssh root@<NODE_IP> -p 20002
   systemctl restart c4_cloud_command
   systemctl restart c4
   ```

2. **Check network:**
   ```bash
   # From another node
   ping <FAILED_NODE_IP>
   traceroute <FAILED_NODE_IP>
   ```

3. **Check hardware:**
   - Review system logs: `dmesg | tail -100`
   - Check for hardware errors
   - Consider node replacement if hardware failed

### Reserve Node Not Taking Over

**Symptom**: Active node fails, but reserve node doesn't take over

**Checks:**
```bash
# Check reserve nodes
confd_client node_list | grep reserve

# Check fail safety configuration
confd_client db_info db_name: MY_DATABASE | grep -i fail

# Check eventd log
tail -100 /exa/logs/logd/eventd.log | grep -i failover
```

**Common causes:**
- No reserve nodes configured
- Reserve nodes also failed
- Fail safety not configured correctly

**Solutions:**
1. **Verify reserve nodes exist:**
   ```bash
   confd_client node_list
   # Look for nodes not in the active node list
   ```

2. **Check fail safety settings:**
   ```bash
   grep -A 50 "^\[DB :" /exa/etc/EXAConf | grep -i fail
   ```

3. **Manually trigger failover:**
   ```bash
   # If automatic failover didn't work, you may need to manually
   # activate the reserve node (contact Exasol support)
   ```

### Node Stuck in Stage 'i' or 'c'

**Symptom**: Node doesn't progress through deployment stages

**Stage meanings:**
- **i**: Image deployed (initial)
- **c**: COS running (cluster OS active)
- **d**: Database running (fully operational)

**Checks:**
```bash
# Check c4 deployment status
c4 ps

# Check on the node
ssh root@<NODE_IP> -p 20002
systemctl status c4
systemctl status c4_cloud_command

# Check logs
tail -100 /exa/logs/cored/exainit.log
tail -100 /exa/logs/logd/ConfD.log
```

**Common causes:**
- Services not starting
- Database not starting
- Configuration issues

**Solutions:**
1. **If stuck at 'i'**: Services haven't started
   ```bash
   systemctl start c4
   systemctl start c4_cloud_command
   ```

2. **If stuck at 'c'**: Database hasn't started
   ```bash
   confd_client db_start db_name: MY_DATABASE
   ```

### Node Replacement Failed

**Symptom**: Node replacement via c4 fails

**Checks:**
```bash
# Check c4 output for errors
./c4 host play -i config

# Check SSH connectivity
ssh exasol@<NEW_NODE_IP>

# Check configuration
cat config | grep -E "CCC_HOST|CCC_PLAY"
```

**Common causes:**
- SSH authentication failed
- Wrong IP address in config
- Version mismatch
- Network issues

**Solutions:**
1. **Fix SSH access:**
   ```bash
   # Copy SSH key to new node
   ssh-copy-id exasol@<NEW_NODE_IP>
   ```

2. **Verify configuration:**
   ```bash
   # Check IPs match
   grep CCC_HOST_ADDRS config
   grep CCC_HOST_SETUP_ADDRS config
   
   # Check version
   grep CCC_PLAY_WORKING_COPY config
   confd_client version_list
   ```

3. **Check network:**
   ```bash
   ping <NEW_NODE_IP>
   telnet <NEW_NODE_IP> 22
   ```

---

## Best Practices

### Planning Node Operations

**1. Always plan node operations during maintenance windows**
- Schedule downtime
- Notify users
- Have rollback plan

**2. Stop the database before node operations**
- Prevents data corruption
- Ensures clean state
- Required for many operations

**3. Have monitoring in place**
- Monitor node health continuously
- Set up alerts for node failures
- Track key metrics (CPU, memory, disk)

**4. Document node configurations**
- Keep c4 config files in version control
- Document IP addressing scheme
- Track which nodes have what roles

**5. Test procedures in non-production first**
- Add/replace nodes in test environment
- Verify failover behavior
- Measure downtime

### Operating Node Clusters

**1. Use reserve nodes for fail safety**
- Configure at least one reserve node
- Use k=2 for critical production systems
- Test failover regularly

**2. Monitor node health proactively**
- Don't wait for failures
- Track trends (disk filling up, memory increasing)
- Replace hardware showing signs of failure

**3. Keep nodes homogeneous**
- Use same hardware specs for all nodes
- Same CPU, memory, disk configuration
- Simplifies management and performance tuning

**4. Use consecutive IP addressing**
- Makes cluster expansion easier
- Simplifies troubleshooting
- Clearer network topology

**5. Maintain consistent configuration**
- Use same c4 config file for all operations
- Keep config in version control
- Document any manual changes

### Maintenance and Operations

**1. Regular maintenance schedule**
- OS updates: Coordinate across nodes
- Hardware checks: Proactive replacement
- Log cleanup: Prevent disk full

**2. Backup before major changes**
- Backup database before adding/removing nodes
- Keep EXAConf backups: `/exa/etc/EXAConf.*`
- Document current state

**3. Test failover regularly**
- Suspend an active node
- Verify reserve node takes over
- Measure failover time

**4. Clean up logs periodically**
- Old logs consume disk space
- Use log rotation
- Archive old logs to external storage

**5. Keep node assignments clear**
- Document which nodes are active vs reserve
- Track node roles (data, compute)
- Update documentation after changes

### Capacity Planning

**1. Add nodes before running out of capacity**
- Monitor disk usage trends
- Plan additions 3-6 months in advance
- Consider future growth

**2. Add nodes in pairs**
- Maintains balance
- Simplifies planning
- Better distribution

**3. Consider redundancy level**
- Higher k = more availability, more cost
- Balance cost vs requirements
- Document SLA requirements

**4. Plan for peak loads**
- Add capacity for peak, not average
- Consider seasonal variations
- Test performance after adding nodes

### Troubleshooting Preparation

**1. Have emergency contacts**
- Exasol support contact info
- Hardware vendor contacts
- Network team contacts

**2. Document recovery procedures**
- Node replacement steps
- Failover procedures
- Rollback procedures

**3. Keep diagnostic tools ready**
- SSH access to all nodes
- c4 configuration files
- Monitoring dashboards

**4. Regular health checks**
- Weekly node status review
- Monthly performance analysis
- Quarterly capacity review

---

## Related Documentation

- [ConfD](https://docs.exasol.com/db/latest/confd/confd.htm)
- [Exasol Deployment Tool (c4)](https://docs.exasol.com/db/latest/administration/on-premise/admin_interface/c4.htm)
- [Node Jobs](https://docs.exasol.com/db/latest/confd/overview_node_jobs.htm)
- [Fail Safety](https://docs.exasol.com/db/latest/planning/fail_safety.htm)
- [System Requirements](https://docs.exasol.com/db/latest/administration/on-premise/installation/system_requirements.htm)
- [Exasol Directory Structure](./exasol_directory_structure.md)
- [c4 Comprehensive Guide](./c4_comprehensive_guide.md)
- [ConfD Client Commands](./confd_client/)

## Common Questions

- How do I add nodes to an Exasol cluster?
- How do I replace a failed node in Exasol?
- How do I stop and start Exasol nodes safely?
- What are reserve nodes in Exasol?
- How does Exasol failover work?
- How do I suspend a node for maintenance?
- How do I check if a node is online?
- What is the difference between active and reserve nodes?
- How do I activate reserve nodes as data nodes?
- How many reserve nodes do I need?
- What happens if an active node fails?
- How do I add capacity to my Exasol cluster?
- How do I prepare a host for a new Exasol node?
- What IP addresses do I need for new nodes?
- How do I verify nodes were added successfully?
- Why won't my node start?
- How do I troubleshoot offline nodes?
- What logs should I check for node issues?
- How do I monitor node health in Exasol?
- What is the REORGANIZE command used for?
- How long does it take to add nodes?
- Can I add multiple nodes at once?
- Do I need to stop the database to add reserve nodes?
- What is hot standby in Exasol?
- How do I configure fail safety?
- What is the difference between stage 'c' and 'd'?
- How do I check which nodes are active vs reserve?
- Can I rename nodes in Exasol?
- How do I remove nodes from a cluster?
- What is node_suspend used for?

## Summary

**Key Operations:**

✅ **Stop/Start Nodes**: Always stop database first, manage services with systemctl  
✅ **Add Data Nodes**: Reserve IPs → Add to deployment → Reserve → Append to volume → Stop DB → Enlarge → Start → REORGANIZE  
✅ **Add Reserve Nodes**: Same as data nodes but stop after appending to volume  
✅ **Replace Nodes**: Stop DB → Prepare new host with same IPs → Update config → Deploy → Start DB  
✅ **Suspend/Resume**: Lightweight operation for maintenance, automatic resume on startup  

**Critical Commands:**
- `c4 ps` - Check deployment status
- `c4 connect -i <PLAY_ID> -s cos` - Connect to COS
- `confd_client node_list` - List all nodes
- `confd_client db_stop db_name: <DB>` - Stop database
- `confd_client db_start db_name: <DB>` - Start database
- `systemctl status c4` - Check c4 service
- `systemctl status c4_cloud_command` - Check COS service

**File Paths:**
- Configuration: `/exa/etc/EXAConf`
- Logs: `/exa/logs/logd/`, `/exa/logs/cored/`, `/exa/logs/db/`
- Jobs: `/exa/spool/jobs/`

**Best Practices:**
- Plan operations during maintenance windows
- Always stop database before major operations
- Use reserve nodes for fail safety
- Monitor node health proactively
- Keep documentation updated
- Test in non-production first
