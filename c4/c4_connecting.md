---
tool_name: c4
doc_type: guide
category: c4 Connecting
title: "c4 Connecting to Deployments"
summary: "c4 provides flexible connection options to access different subsystems within a deployment."
---
# c4 Connecting to Deployments

## Overview

c4 provides flexible connection options to access different subsystems within a deployment.

## Connection Syntax

**Option 1: Using play ID and options**
```bash
c4 connect -i PLAY_ID [-n NODE] [-s SUBSYSTEM]
```

**Option 2: Using shorthand notation**
```bash
c4 connect -t NUM[.NODE][/SUBSYSTEM]
```

## Connection Parameters

| Parameter | Description |
|-----------|-------------|
| **PLAY_ID** | Unique deployment identifier (safest method)<br>Example: `c3275f84` |
| **NUM** | Deployment number (N in `c4 ps`)<br>**Warning**: Dynamic, may change |
| **NODE** | Cluster node ID (NODE in `c4 ps`)<br>If omitted, connects to first active node |
| **SUBSYSTEM** | Target subsystem: `cos`, `host`, or `db`<br>If omitted, connects to database (if online) |

## Subsystems

| Subsystem | Description | Connection Examples |
|-----------|-------------|---------------------|
| **/host** | Host operating system | `c4 connect -i c3275f84 -n 11 -s host`<br>`c4 connect -t 1.11/host`<br>`c4 connect -t 1/host` |
| **/cos** | Cluster Operating System (COS) | `c4 connect -i c3275f84 -s cos`<br>`c4 connect -t 1/cos` |
| **/db** | Database (built-in SQL client) | `c4 connect -i c3275f84 -s db`<br>`c4 connect -i c3275f84`<br>`c4 connect -t 1/db`<br>`c4 connect -t 1` |

## Connection Examples

### Connect to database (default)
```bash
# Using play ID
c4 connect -i c3275f84

# Using deployment number
c4 connect -t 1
```

### Connect to specific node
```bash
# Node 11 in deployment c3275f84
c4 connect -i c3275f84 -n 11

# Using shorthand
c4 connect -t 1.11
```

### Connect to COS
```bash
# Run ConfD commands
c4 connect -i c3275f84 -s cos

# Using shorthand
c4 connect -t 1/cos
```

### Connect to host OS
```bash
# Access host shell
c4 connect -i c3275f84 -n 11 -s host

# Using shorthand
c4 connect -t 1.11/host
```

## Disconnect

To disconnect from the deployment:
```bash
Control+D
```

Or type:
```bash
exit
```

## Single-Line Command Syntax

Execute commands directly without entering interactive session.

**Syntax**:
```bash
c4 connect -i PLAY_ID -s SUBSYSTEM 'COMMAND'
c4 connect -i PLAY_ID -s SUBSYSTEM -- 'COMMAND'
c4 connect -i PLAY_ID -s SUBSYSTEM <<< 'COMMAND'
c4 connect -i PLAY_ID -s SUBSYSTEM < script.sh
```

**When to use `--`**: When passed command includes CLI options that could be parsed by `c4 connect`.

### Examples

**Simple command (no options)**:
```bash
c4 connect -i c3275f84 -s host 'ls'
```

**Command with options** (use `--`):
```bash
c4 connect -i c3275f84 -s host -- 'ls -t'
```

**ConfD commands**:
```bash
c4 connect -i c3275f84 -s cos -- 'confd_client db_stop db_name: MY_DATABASE'
c4 connect -i c3275f84 -s cos -- 'confd_client db_start db_name: MY_DATABASE'
c4 connect -i c3275f84 -s cos -- 'confd_client db_state db_name: MY_DATABASE'
```

**Using stdin with `<<<`**:
```bash
c4 connect -i c3275f84 -s host <<< 'df -h'
```

**Execute script**:
```bash
c4 connect -i c3275f84 -s host < maintenance.sh
```

**Note**: When using script files, do **NOT** prefix commands with `--`.

## Monitor Deployment During Creation

If you connect using `c4 connect -i PLAY_ID` during an ongoing deployment, c4 will output the deployment log in the terminal. This is useful for troubleshooting.

## Built-in SQL Client

The built-in SQL client accessed via `/db` subsystem **always runs in autocommit mode**.

**Example SQL session**:
```bash
c4 connect -i c3275f84 -s db
# Now in SQL client
SELECT * FROM EXA_METADATA;
SELECT CURRENT_TIMESTAMP;
```

## Best Practices

**Use play ID instead of deployment number**
- Play ID is permanent and unique
- Deployment number (N) can change

**Use single-line syntax for automation**
- Scripting and automation
- CI/CD pipelines
- Reduces manual errors

**Choose appropriate subsystem**
- `/db` for SQL queries
- `/cos` for ConfD commands (db operations)
- `/host` for system-level tasks

## Troubleshooting

### Issue: "Permission denied" when connecting

**Cause**: SSH key authentication failure

**Solutions**:
1. Verify SSH key path in configuration
2. Check SSH key permissions (should be 600)
3. Ensure correct key is used for deployment

## Related Documentation

- [c4 Overview](c4_overview.md)
- [c4 Monitoring Deployments](c4_monitoring.md)
- [c4 Managing Nodes](c4_managing_nodes.md)
- [ConfD Client Commands](../confd_client_overview.md)
