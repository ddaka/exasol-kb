---
tool_name: c4
doc_type: reference
category: c4 Deployment Management
title: "c4 Deployment Management"
summary: "`c4` is Exasol's deployment and orchestration tool that manages the cluster lifecycle. It runs on the **host machine** (not in the COS namespace) and provides c..."
---
# c4 Deployment Management

## Overview

`c4` is Exasol's deployment and orchestration tool that manages the cluster lifecycle. It runs on the **host machine** (not in the COS namespace) and provides cluster deployment, node provisioning, working copy distribution, and deployment status monitoring.

## Prerequisites

- SSH access to Exasol host machine (standard SSH port 22, not port 20002)
- The `c4` service must be running (`systemctl --user status c4`)

## Check Deployment Status

```bash
c4 ps
```

Example output:

```
     N  PLAY_ID   NODE  MEDIUM  INSTANCE  DB_VERSION  EXTERNAL_IP  INTERNAL_IP  STAGE  STATE  UPTIME    TTL
 1  09ab18b2  11    host    -         8.33.0      10.70.0.171  10.70.0.171  d      -      105 days  +∞
 2  09ab18b2  11    local   -         8.33.0      -            10.70.0.171  d      -      105 days  +∞
```

## Column Reference

| Column | Description |
|--------|-------------|
| **N** | Entry number |
| **PLAY_ID** | Unique deployment session identifier |
| **NODE** | Node number (11, 12, 13, etc.) |
| **MEDIUM** | `host` (physical/VM), `local` (COS namespace), `cloud` (AWS/Azure/GCP) |
| **INSTANCE** | Cloud instance ID (cloud deployments only) |
| **DB_VERSION** | Exasol database version installed |
| **EXTERNAL_IP** | Public IP address |
| **INTERNAL_IP** | Private/cluster IP address |
| **STAGE** | Deployment stage (see below) |
| **STATE** | Current state (`-` means running normally) |
| **UPTIME** | How long the instance has been running |
| **TTL** | Time-to-live (`+∞` means no expiration) |

## Deployment Stages

Stages represent the deployment progression from cloud resource allocation to a fully operational database:

| Stage | Name | Description |
|-------|------|-------------|
| **a** | Cloud | Allocating cloud resources (AWS, Azure, GCP). Node not yet reachable. |
| **a1** | Cloud substage | Node created but c4 service not reachable. SSH available on port 22. |
| **b** | Boot | Running startup scripts and system configuration. C4 service is reachable. |
| **b1** | Boot substage | Fetching working copies. SSH available on port 20002 (COS namespace). |
| **c** | COS | COS is initialized and reachable. Management tools available. Database not yet started. |
| **d** | Database | Database is operational. Cluster fully functional. |

**Normal operational state**: `STAGE=d, STATE=-` — cluster is fully operational.

## Common c4 Commands

```bash
# Show current deployments
c4 ps

# Show all deployments including stopped
c4 ps -a

# Show detailed IP and database information
c4 ps -V ip,db

# Show specific deployment by PLAY_ID
c4 ps 09ab18b2

# Get deployment status summary
c4 status

# Query deployment data using jq-style expressions
c4 ps -e '.[].play.id'

# Output deployments in JSON format
c4 ps -o json
```

## Common Workflows

### Check Overall Cluster Health

```bash
ssh ddaka@10.70.0.171

# Check services are running
systemctl --user status c4
systemctl --user status c4_cloud_command

# Check deployment status — all nodes should be in stage 'd'
c4 ps
```

### Monitor Deployment Progress

```bash
# Watch deployment status in real-time
watch -n 2 'c4 ps'
```

## Troubleshooting

### Deployment Stuck in Stage 'b'

`c4 ps` shows `STAGE=b` for an extended time.

- Check logs: `journalctl --user -u c4_cloud_command -f`
- Verify network connectivity
- Check disk space: `df -h`
- Ensure working copies are accessible
- Review c4 logs: `journalctl --user -u c4`

### Deployment Shows Unexpected STATE

If `STATE` shows anything other than `-`, check:

- Service logs: `journalctl --user -u c4_cloud_command -n 200`
- C4 server logs: `journalctl --user -u c4 -n 200`
- Disk and memory availability
