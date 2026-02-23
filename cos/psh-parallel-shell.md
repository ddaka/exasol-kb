---
tool_name: psh
doc_type: reference
category: Parallel Shell
title: "psh — Parallel Shell for Exasol Clusters"
summary: "`psh` (Parallel Shell) executes shell commands in parallel across all nodes in an Exasol cluster. It is the primary tool for running commands, checking status,..."
---
# psh — Parallel Shell for Exasol Clusters

## Overview

`psh` (Parallel Shell) executes shell commands in parallel across all nodes in an Exasol cluster. It is the primary tool for running commands, checking status, and performing diagnostics cluster-wide.

**Location**: `/opt/exasol/cos-<version>/bin/psh`

**Context**: Must be executed **inside the COS namespace** (SSH port 20002), not on the host machine.

## Prerequisites

- SSH access to Exasol COS namespace: `ssh root@<host> -p 20002`
- Exasol COS v8.0 or later

## Basic Syntax

```bash
psh "<command>"
```

Output is prefixed with a node identifier (e.g., `0011:`) followed by the command result.

## Examples

### System Status

```bash
# Check hostname on all nodes
psh "hostname"
# Output:
# 0011: n11
# 0012: n12

# Check disk space
psh "df -h /"

# Check memory usage
psh "free -h"

# View network configuration
psh "ip addr show"
```

### Exasol-Specific Checks

```bash
# Check EXAConf settings on all nodes
psh "cat /exa/etc/EXAConf | grep -A5 'Node = '"

# Verify EXAConf is identical across nodes
psh "md5sum /exa/etc/EXAConf"

# Check COS version on all nodes
psh "cat /opt/exasol/cos-*/VERSION"

# Check database version
psh "cat /opt/exasol/db-*/VERSION"

# Count Exasol processes on each node
psh "ps aux | grep -E '(exasol|dwad)' | grep -v grep | wc -l"
```

### File Operations

```bash
# Create a file on all nodes
psh "touch /tmp/test.txt"

# List files
psh "ls -lh /tmp/*.log"

# Clean up temporary files
psh "rm -f /tmp/*.tmp"
```

### Log Collection

```bash
# Copy logs to collection directory on each node
psh "mkdir -p /tmp/log_collection"
psh "cp /exa/logs/cored/exainit.log /tmp/log_collection/exainit_\$(hostname).log"

# Create archive on each node
psh "tar czf /tmp/logs_\$(hostname).tar.gz /tmp/log_collection/"
```

## psh vs dwad_client

- **psh**: Runs commands on each node individually, output is per-node.
- **dwad_client**: Returns cluster-wide database info (same result from any node). No need to wrap `dwad_client list` in psh.

```bash
# Cluster-wide data — same from any node, no psh needed
dwad_client list

# Per-node information — use psh
psh "hostname"
```

## Important Notes

- Commands must be **quoted** to ensure proper execution
- Commands run with the same privileges as the user executing `psh`
- In **single-node clusters**, `psh` only executes on the current node
- Use **caution with destructive commands** — they affect all nodes simultaneously
- Avoid interactive commands (commands that wait for user input)

## Best Practices

1. **Test on single node first** before running cluster-wide
2. **Quote commands properly**: `psh "command here"`
3. **Use dry-run mode** before destructive operations (e.g., `ls` before `rm`)
4. **Use md5sum** to verify file consistency across nodes
5. **Avoid syncing sensitive credentials** in plaintext

## Troubleshooting

### psh Shows Output from Only One Node

- This is normal in single-node clusters
- Check cluster status: `dwad_client list`
- Verify node states: `confd_client node_list`
- Check node connectivity: `psh "ping -c 1 n11"`
- Suspended nodes are normal for reserve nodes

### Command Hangs or Doesn't Complete

- Press `Ctrl+C` to cancel
- Check if nodes are responsive: `confd_client node_list`
- Try a simpler command first: `psh "echo test"`
- Avoid commands that wait for interactive input
