---
tool_name: cos
doc_type: reference
category: COS Service Management
title: "COS systemd Service Management"
summary: "Exasol COS runs as two systemd services on the host machine. Managing these services is essential for starting, stopping, and troubleshooting the Exasol cluster..."
---
# COS systemd Service Management

## Overview

Exasol COS runs as two systemd services on the host machine. Managing these services is essential for starting, stopping, and troubleshooting the Exasol cluster.

## Services

### c4.service — Deployment Server

Manages cluster lifecycle and orchestration: deployments, updates, node provisioning, and working copy distribution.

### c4_cloud_command.service — COS Namespace Manager

Creates and manages the Linux namespace that runs the Exasol cluster. Handles database processes, confd, dwad, bucketfsd, and other COS services.

**Warning**: Restarting `c4_cloud_command` **stops all databases** and restarts the entire COS environment.

## Check Service Status

**Rootless installations (standard)**:

```bash
systemctl --user status c4
systemctl --user status c4_cloud_command
```

**Root-based installations**:

```bash
systemctl status c4
systemctl status c4_cloud_command
```

Most Exasol deployments use rootless installations where services run under a user account. Use `systemctl --user` in those cases.

## Start, Stop, and Restart Services

**Rootless installations**:

```bash
# Restart COS namespace (WARNING: stops all databases!)
systemctl --user restart c4_cloud_command

# Restart c4 deployment server
systemctl --user restart c4

# Stop COS namespace
systemctl --user stop c4_cloud_command

# Start COS namespace
systemctl --user start c4_cloud_command
```

**Root-based installations**:

```bash
systemctl restart c4_cloud_command
systemctl restart c4
systemctl stop c4_cloud_command
systemctl start c4_cloud_command
```

## View Service Logs

**Rootless installations**:

```bash
# Follow COS namespace logs in real-time
journalctl --user -u c4_cloud_command -f

# Follow c4 server logs
journalctl --user -u c4 -f

# View last 100 lines
journalctl --user -u c4_cloud_command -n 100

# Search for errors in the last hour
journalctl --user -u c4_cloud_command --since "1 hour ago" | grep -i error

# View logs from a specific date range
journalctl --user -u c4_cloud_command --since "2026-02-01" --until "2026-02-02"
```

**Root-based installations**:

```bash
journalctl -u c4_cloud_command -f
journalctl -u c4 -f
```

## Restart COS After Configuration Change

```bash
ssh ddaka@10.70.0.171

# Check current state
c4 ps

# Restart COS namespace (WARNING: stops databases!)
systemctl --user restart c4_cloud_command

# Monitor restart progress
watch -n 2 'c4 ps'

# Verify stage returns to 'd' (database operational)
```

## Check COS Namespace Processes

After accessing the COS namespace (`ssh root@<host> -p 20002`):

```bash
# Check running Exasol services
ps aux | grep -E '(confd|dwad|bucketfsd|exasol)'

# Check database processes
ps aux | grep controller

# Check storage services
ps aux | grep cos_storage
```

## Troubleshooting

### Services Won't Start

`systemctl --user start c4_cloud_command` fails.

- Check logs: `journalctl --user -u c4_cloud_command -n 100`
- Verify user has lingering enabled: `loginctl show-user $USER`
- Check for port conflicts (20002, 8563, 443)
- Ensure sufficient resources (memory, disk space)

### Cannot Access COS Namespace

`ssh root@<host> -p 20002` connection refused.

- Check service is running: `systemctl --user status c4_cloud_command`
- Verify deployment stage: `c4 ps` (needs STAGE >= b1)
- Check firewall rules for port 20002
- Review logs: `journalctl --user -u c4_cloud_command | grep -i ssh`

### Service Shows Failed Status

`systemctl --user status c4_cloud_command` shows "failed".

- View failure details: `systemctl --user status c4_cloud_command -l`
- Check recent logs: `journalctl --user -u c4_cloud_command -n 200`
- Reset failed state: `systemctl --user reset-failed c4_cloud_command`
- Try manual start: `systemctl --user start c4_cloud_command`
