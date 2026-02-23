---
tool_name: cos
doc_type: concept
category: COS Architecture
title: "Exasol COS Architecture and Access Methods"
summary: "Exasol COS (Cluster Operating System) runs as a systemctl service on the host machine within a Linux namespace. This two-layer architecture separates host-level..."
---
# Exasol COS Architecture and Access Methods

## Overview

Exasol COS (Cluster Operating System) runs as a systemctl service on the host machine within a Linux namespace. This two-layer architecture separates host-level management from the cluster environment.

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────┐
│  Host Machine (Linux Server) — SSH port 22              │
│  ├─ systemd services:                                   │
│  │  ├─ c4.service (deployment server)                   │
│  │  └─ c4_cloud_command.service (COS namespace manager) │
│  │                                                       │
│  └─ COS Namespace (Linux namespace) — SSH port 20002    │
│     ├─ Cluster management tools:                        │
│     │  ├─ psh (parallel shell)                          │
│     │  ├─ cos_sync_files (file sync)                    │
│     │  ├─ confd_client (configuration)                  │
│     │  ├─ dwad_client (database management)             │
│     │  ├─ sdfs (distributed file system)                │
│     │  └─ exainit (initialization)                      │
│     │                                                    │
│     └─ Database processes (EXASolution, controller)     │
└─────────────────────────────────────────────────────────┘
```

## Two Access Methods

### 1. Host Machine Access (SSH port 22)

Connect to the underlying Linux host system:

```bash
ssh ddaka@10.70.0.171
```

Available on the host:
- Standard Linux tools and utilities
- `systemctl` for service management (`c4`, `c4_cloud_command`)
- `c4` deployment commands (`c4 ps`, `c4 status`)
- `journalctl` for service logs

### 2. COS Namespace Access (SSH port 20002)

Connect directly to the Exasol COS namespace:

```bash
ssh root@10.70.0.171 -p 20002
```

Available in the namespace:
- `psh` — parallel shell execution across nodes
- `cos_sync_files` — file synchronization across nodes
- `confd_client` — cluster configuration management
- `dwad_client` — database management
- `sdfs` — distributed file system operations
- `exainit` — cluster initialization
- Database logs at `/exa/logs/`
- Cluster configuration at `/exa/etc/EXAConf`

**Critical**: Tools like `psh`, `cos_sync_files`, `confd_client`, `dwad_client`, `sdfs`, and `exainit` **only work inside the COS namespace**, not on the host machine.

## Host Machine Command Reference

```bash
# C4 deployment status
c4 ps
c4 ps -a
c4 ps -V ip,db
c4 status

# Service management (rootless — standard)
systemctl --user status c4
systemctl --user status c4_cloud_command
systemctl --user restart c4_cloud_command

# Service management (root-based installations)
systemctl status c4
systemctl status c4_cloud_command

# Service logs
journalctl --user -u c4_cloud_command -f
journalctl --user -u c4 -f
```

## COS Namespace Command Reference

```bash
# Cluster management
confd_client node_list
confd_client db_list
dwad_client list

# Parallel execution and file sync
psh "<command>"
cos_sync_files <path>

# SDFS file operations
sdfs shortlist v0001
sdfs getraw v0001 path/to/file
sdfs addraw v0001 local_file path/in/sdfs

# Configuration and logs
vi /exa/etc/EXAConf
tail -f /exa/logs/cored/exainit.log
tail -f /exa/logs/logd/database_name.log
```

## Key Ports

| Port  | Purpose                        |
|-------|--------------------------------|
| 22    | SSH to host machine            |
| 20002 | SSH to COS namespace           |
| 8563  | Exasol database connections    |
| 443   | HTTPS / AdminUI           |
| 2021  | SDFS FTP/SFTP access           |

## Important Notes

- Most Exasol deployments use **rootless installations** — use `systemctl --user` for service management.
- Restarting `c4_cloud_command` **stops all databases** and restarts the entire COS environment.
- The COS namespace is created and managed by the `c4_cloud_command` systemd service.
