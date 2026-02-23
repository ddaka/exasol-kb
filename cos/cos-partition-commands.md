---
tool_name: cos
doc_type: reference
category: Partition and Node Commands
---

# COS Partition and Node Commands

## Overview

COS (Cluster Operating System) provides commands for managing partitions, nodes, and cluster state. Partitions are the fundamental unit of process isolation in COS, and nodes are the physical machines in the cluster.

All commands run inside the COS namespace (SSH port 20002).

## Core Partition Commands

### cosps — List Partitions

Show information about partitions in the COS cluster.

**Basic Usage**:

```bash
cosps              # List all partitions
cosps -f           # Show full information
cosps -m           # Show user/group names instead of IDs
cosps -e           # Show partition environment variables
```

**Filtering**:

```bash
cosps -p 56832     # Children of specific partition
cosps -u 500       # Partitions owned by specific user
cosps -g 500       # Partitions by group
```

**Display**:

```bash
cosps -N           # Show physical node IDs
cosps -r           # Show offline nodes
cosps -n           # Show root nodes
cosps -q           # Quiet output (no headers)
```

**Common Combinations**:

```bash
cosps -f -m              # Full details with names
cosps -p 27 -m           # DWAD's children (database components)
cosps -u 500 -e          # User partitions with environment
```

**Example Output**:

```
ID    OWNER GROUP PARENT FLAGS ONLINE NODES   PHYSICAL NODES  COMMAND
16    0     0     0      R--I  11              -               sshd
27    0     0     0      RA-I  11              -               dwad
56832 500   500   27     --EK  11              -               controller-Exasol
```

### cosexec — Execute in Partition

Execute commands within a specific partition's context.

```bash
cosexec <partition-id> -- <command> [args...]
```

**Examples**:

```bash
cosexec 56832 -- ls -la /exa/logs/db/Exasol/
cosexec 56840 -- ps aux | grep exasql
cosexec 56832 -- env | grep EXA
cosexec 56832 -- tail -f /exa/logs/db/Exasol/cored_Exasol_*.log
```

### coskill — Terminate Partition

Send signals to kill or stop partitions.

```bash
coskill <partition-id>          # Default kill
coskill -9 <partition-id>       # Force kill (SIGKILL)
coskill -s TERM <partition-id>  # Send specific signal
```

### coswatch — Monitor Partition Events

Watch partition lifecycle events in real-time.

```bash
coswatch <partition-id>         # Monitor specific partition
coswatch <partition-id> &       # Monitor in background
```

### cospstree — Partition Hierarchy Tree

Display partition parent-child relationships.

```bash
cospstree              # Full tree
cospstree 27           # Tree from partition 27
cospstree -p 56832     # Show ancestors
```

### cosconnect — Connect to Partition

Enter a partition context interactively.

```bash
cosconnect <partition-id>
```

### cosexit — Exit Partition

Leave partition context.

```bash
cosexit
```

## Node Management Commands

### cosadd — Add Node to Partition

```bash
cosadd -N <node-name> <partition-id>    # Add node
cosadd -N <node-name> -S <partition-id> # Add node in suspended state
cosadd -N <node-name> -x               # Add and extend existing partitions
```

### cosrm — Remove Node from Partition

```bash
cosrm -n <node-id> <partition-id>   # Remove node
cosrm -n <node-id> -f               # Force removal
```

### cosmv — Move Partition to Node

```bash
cosmv -N <target-node> <partition-id>
```

### cosmod — Modify Partition Attributes

```bash
cosmod -r <partition-id>   # Enable auto-restart
cosmod -a <partition-id>   # Enable auto-add
cosmod -d <partition-id>   # Remove flag
```

## Cluster Information Commands

### cosinfo — Cluster Information

```bash
cosinfo -n    # Current node ID
cosinfo -i    # Node name/hostname
cosinfo -p    # Current partition ID
cosinfo -c    # Cluster ID
cosinfo -N    # Partition node list
cosinfo -e    # All existing nodes
```

**Scripting**:

```bash
NODE_ID=$(cosinfo -n)
echo "Current node: $NODE_ID"

if PART_ID=$(cosinfo -p 2>/dev/null); then
    echo "Running in partition $PART_ID"
fi
```

### coslookup — Node Lookup

```bash
coslookup -A --json            # All nodes as JSON
coslookup -n 11 --get-address  # Specific node address
coslookup -A -o --get-name     # All online node names
coslookup -a 10.70.0.171 --get-id  # Find node by IP
coslookup -n 11 --get-uuid     # Get node UUID
```

### cosnodeinfo — Node Details

```bash
cosnodeinfo -l       # List all nodes
cosnodeinfo -n 11    # Specific node details
cosnodeinfo -v       # Verbose information
```

### coswait — Wait for Cluster Condition

```bash
coswait -o 4           # Wait for 4 nodes online
coswait -c 4           # Wait for cluster size 4
coswait -o 4 -t 300    # Wait with 5-minute timeout
```

### coscheck — Check Cluster Health

```bash
coscheck        # Basic health check
coscheck -s     # Silent mode (exit code only)
coscheck -v     # Verbose output
```

Exit code 0 means healthy; non-zero means issues detected.

## Practical Command Combinations

### Database Health Check

```bash
#!/bin/bash
echo "=== DWAD Status ==="
cosps | grep dwad

echo -e "\n=== Database Controllers ==="
cosps -p 27 -m

echo -e "\n=== Database Components ==="
DB_CONTROLLER=$(cosps | awk '/controller-Exasol/ {print $1}')
if [ -n "$DB_CONTROLLER" ]; then
    cosps -p $DB_CONTROLLER
    COMPONENT_COUNT=$(cosps -p $DB_CONTROLLER | tail -n +2 | wc -l)
    echo "Component count: $COMPONENT_COUNT (expected: 7)"
else
    echo "No database controller found"
fi

echo -e "\n=== Database State ==="
confd_client db_state db: Exasol -j
```

### Monitor Database Startup

```bash
#!/bin/bash
echo "Starting database..."
confd_client db_start db: Exasol -S

while true; do
    clear
    date
    echo "=== Controllers ==="
    cosps -p 27 -m
    DB_CONTROLLER=$(cosps | awk '/controller-Exasol/ {print $1}')
    if [ -n "$DB_CONTROLLER" ]; then
        echo -e "\n=== Components ==="
        cosps -p $DB_CONTROLLER
    fi
    sleep 2
done
```

### Cluster Node Summary

```bash
#!/bin/bash
echo "=== Cluster Nodes ==="
coslookup -A --json | jq -r '.nodes[] | "\(.id)\t\(.name)\t\(.address)\t\(.state)"'

echo -e "\n=== Node IDs ==="
echo "Current node: $(cosinfo -n)"
echo "Current hostname: $(cosinfo -i)"

echo -e "\n=== Partition Distribution ==="
cosps -N | head -20
```

## Command Cheat Sheet

| Command | Purpose | Example |
|---------|---------|---------|
| `cosps -m` | List partitions with names | `cosps -m` |
| `cosps -p 27` | Show database instances | `cosps -p 27 -m` |
| `cosexec` | Run command in partition | `cosexec 56832 -- ls` |
| `cospstree` | Show hierarchy | `cospstree 27` |
| `coswatch` | Monitor events | `coswatch 56832 &` |
| `cosinfo -n` | Current node ID | `cosinfo -n` |
| `coslookup -A -j` | Node info JSON | `coslookup -A --json` |
| `coscheck` | Cluster health | `coscheck -v` |
| `cosps -r` | Offline nodes | `cosps -r -N` |

## Best Practices

- Use `cosps -m` for readability (names instead of IDs)
- Check before killing: `cosps -f | grep <id>` and `cospstree <id>` to see children
- Monitor long operations: `coswatch <partition-id> &`
- Use JSON for scripting: `coslookup -A --json | jq '.nodes[]'`
- Prefer `confd_client` for database operations instead of `coskill`
