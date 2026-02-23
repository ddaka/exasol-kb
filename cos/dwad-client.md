---
tool_name: dwad_client
doc_type: reference
category: Database Watchdog
---

# dwad_client — Database Watchdog Daemon Client

## Overview

`dwad_client` is the direct interface to the Database Watchdog Daemon (DWAD). It provides low-level database management commands for advanced administration and troubleshooting.

**Note**: Most database operations should use `confd_client` instead. Use `dwad_client` for advanced administration, troubleshooting, and operations not available through `confd_client`.

All commands run inside the COS namespace (SSH port 20002).

## Database Listing

```bash
dwad_client list              # List all databases
dwad_client shortlist         # Short list
```

## Database Operations

### Start

```bash
dwad_client start MyDatabase
dwad_client start-features MyDatabase "feature1,feature2"
dwad_client start-wait MyDatabase                # Start and wait for reachability
dwad_client start-create-new-db MyDatabase       # Start with create-new-db flag
dwad_client start-maintenance MyDatabase         # Start in maintenance mode
dwad_client start-failsafety MyDatabase          # Start for failsafety
```

### Stop

```bash
dwad_client stop MyDatabase
dwad_client stop-wait MyDatabase                 # Stop and wait
dwad_client stop-signal MyDatabase 15 30         # Stop with signal and timeout
dwad_client stop-force MyDatabase                # Stop immediately to setup state
```

### Other

```bash
dwad_client add MyDatabase                       # Add new database
dwad_client del MyDatabase                       # Delete database
dwad_client rename OldName NewName               # Rename database
```

## Setup and Configuration

```bash
dwad_client setup MyDatabase /path/to/setup.conf
dwad_client setup-param MyDatabase max_memory 16GB
dwad_client setup-node-groups "n11,n12" "n13,n14"
dwad_client get-node-groups
dwad_client print-params MyDatabase
dwad_client print-original-setup MyDatabase
dwad_client print-setup MyDatabase
```

## Database Information

```bash
dwad_client uptime MyDatabase                    # Uptime in seconds
dwad_client conn MyDatabase                      # Connection info (host:port)
dwad_client sys-nodes MyDatabase                 # List system nodes
dwad_client db-size-info MyDatabase              # Database size info
dwad_client space-info MyDatabase                # Space info
dwad_client volume-space-info MyDatabase         # Volume space info
dwad_client show-files MyDatabase 0              # Datafiles for iproc
dwad_client pdd-proc MyDatabase                  # PDD process info
dwad_client pdd-proc-wait MyDatabase             # Wait for PDD info
dwad_client flush-pdd-proc MyDatabase            # Flush PDD info
dwad_client bg-restore-state MyDatabase          # Background restore state
dwad_client check-restore-ready-state MyDatabase # Restore ready check
dwad_client volume-restore                       # Volume restore check
```

## Worker and Sub-Database Operations

### Worker Databases

```bash
dwad_client add-workerdb MasterDB WorkerDB
dwad_client setup-workerdb WorkerDB /path/to/setup.conf
dwad_client start-workerdb WorkerDB
dwad_client stop-workerdb WorkerDB
dwad_client del-workerdb WorkerDB
```

### Sub-Databases

```bash
dwad_client add-subdb MasterDB SubDB
dwad_client setup-subdb SubDB /path/to/setup.conf
dwad_client start-subdb SubDB
dwad_client stop-subdb SubDB
dwad_client del-subdb SubDB
```

## Node Management

```bash
dwad_client extend-system MyDatabase                    # Add active node
dwad_client insert-rnode MyDatabase n15                 # Insert reserve node
dwad_client remove-rnode MyDatabase n15                 # Remove reserve node
dwad_client mark-inactive-node MyDatabase n11           # Mark node inactive
dwad_client remove-inactive-node MyDatabase n11         # Remove inactive node
dwad_client switch-nodes MyDatabase n11 n15             # Switch active/reserve
dwad_client start-on-nodes MyDatabase "command"          # Run on all nodes
dwad_client start-on-nodes-wait MyDatabase "command"     # Run and wait
```

## Backup and Restore

### Storage Backup

```bash
dwad_client storage-backup MyDatabase 0 0 1735862400
dwad_client abort-backup MyDatabase
```

### Snapshot

```bash
dwad_client snapshot MyDatabase 0 1735862400
```

### Storage Restore

```bash
dwad_client storage-restore MyDatabase 0 backup_20250101                 # Blocking
dwad_client storage-restore-nonblocking MyDatabase 0 backup_20250101     # Non-blocking
dwad_client storage-restore-virtual MyDatabase 0 backup_20250101         # Virtual
```

### Snapshot Restore

```bash
dwad_client snapshot-restore MyDatabase 0 snap_20250101 1                # Blocking
dwad_client snapshot-restore-nonblocking MyDatabase 0 snap_20250101 1    # Non-blocking
dwad_client snapshot-restore-virtual MyDatabase 0 snap_20250101          # Virtual
```

## Database Volume Operations

### Shrink Database

```bash
dwad_client shrink-db MyDatabase 102400 1 1    # Dry-run: 100GB, persistent
dwad_client shrink-db MyDatabase 102400 1 0    # Execute: 100GB, persistent
dwad_client abort-shrink MyDatabase
dwad_client shrink-status MyDatabase
```

### Defragment Database

```bash
dwad_client defrag-db MyDatabase 80 1    # Dry-run: 80% utilization
dwad_client defrag-db MyDatabase 80 0    # Execute
dwad_client abort-defrag MyDatabase
```

## Password Management

```bash
dwad_client sys-password MyDatabase                     # Get current sys password
dwad_client set-sys-password MyDatabase "newpassword"   # Set new password
```
