---
tool_name: cos_storage
doc_type: reference
category: Storage Commands
---

# COS Storage Commands (CS Tools)

## Overview

CS (COS Storage) tools manage EXAStorage, the distributed storage layer in Exasol. These commands control volumes, HDDs, snapshots, storage performance, and the storage service.

All commands run inside the COS namespace (SSH port 20002).

## csinfo — Storage Information

Display comprehensive storage and volume information.

```bash
csinfo -v                    # Show all volumes
csinfo -v -i <volume-id>    # Specific volume info
csinfo -D                    # Detailed HDD info
csinfo -H                    # HDD status (ONLINE/OFFLINE)
csinfo -V                    # Volume configuration
csinfo -n                    # Node storage information
csinfo -n 11                 # Storage for specific node
csinfo -r --id=<volume-id>  # Volume ranges
```

**Example Output**:

```
VOLUME ID   SIZE    NODES   TYPE    STATUS
10001       50GB    4/4     DATA    ONLINE
10002       100GB   4/4     DATA    ONLINE
10003       20GB    4/4     TEMP    ONLINE
```

## csvol — Volume Operations

Create and manage storage volumes.

```bash
csvol create --size=<size> --type=<type>    # Create volume
csvol delete --id=<volume-id>               # Delete volume
csvol list                                   # List volumes
csvol info --id=<volume-id>                  # Volume details
```

**Examples**:

```bash
csvol create --size=100G --type=data    # 100GB data volume
csvol create --size=50G --type=temp     # 50GB temp volume
csvol delete --id=10005
```

## csresize — Resize Volumes

Resize storage volumes by appending/removing nodes or changing segment sizes.

**Append Nodes**:

```bash
csresize -a -v 0 -m 2                 # Append master nodes by count
csresize -a -v 0 -m 2 -n "13,14"      # Append specific nodes by ID
csresize -a -v 0 -m 2 -N "n13,n14"    # Append nodes by name
```

**Remove Nodes**:

```bash
csresize -r -v 0 -n "11,12"           # Remove nodes by ID
csresize -r -v 0 -N "n11,n12"         # Remove nodes by name
csresize -r -v 0 -n "11,12" -p        # Remove without replacing (purge)
csresize -r -v 0 -n "11" -f           # Force removal
```

**Enlarge/Shrink**:

```bash
csresize -e -v 0 -b 1000              # Enlarge by blocks
csresize -s -v 0 -b 500               # Shrink by blocks
```

**Change Redundancy**:

```bash
csresize -i -v 0 -l 2                 # Increase redundancy
csresize -d -v 0 -l 1                 # Decrease redundancy
```

## cslabel — Manage Volume Labels

```bash
cslabel -a -v 0 -l "production"       # Add label
cslabel -a -v 0 -l "primary" -F       # Add label to front
cslabel -r -v 0 -l "production"       # Remove specific label
cslabel -r -v 0                        # Remove all labels
cslabel -f -l "production"             # Find volumes by label
```

## csmove — Move Volume Nodes/Segments

```bash
csmove -m -v 0 -s "11,12" -d "13,14"  # Move nodes by ID
csmove -m -v 0 -S "n11,n12" -D "n13,n14"  # Move by name
csmove -M -v 0 -i 5 -d "13"           # Move specific segment
csmove -m -v 0 -s "11" -d "13" -f     # Force movement
```

Movement may be denied if segments have snapshot maps, redundancy conflicts, or source is recovering.

## cssetio — Enable/Disable I/O

```bash
cssetio -a 0 -v 0       # Disable application I/O
cssetio -a 1 -v 0       # Enable application I/O
cssetio -i 0 -v 0       # Disable internal I/O
cssetio -i 1 -v 0       # Enable internal I/O
cssetio -a 0 -i 0 -v 0  # Disable both
```

## cssnap — Storage Snapshots

```bash
cssnap -v <volume-id>                       # Create snapshot
cssnap -v <volume-id> --name=<name>         # Named snapshot
cssnap --list                                # List snapshots
cssnap --restore --id=<snapshot-id>          # Restore
cssnap --delete --id=<snapshot-id>           # Delete
```

**Use Cases**: Before major updates, point-in-time recovery, testing changes.

## cshdd — Manage Storage Disks

**Add HDD**:

```bash
cshdd -a -n 11 -h /dev/sdb1 -t standard              # Add HDD
cshdd -a -n 11 -h /dev/sdb1 -t standard -p /etc/cos/sdb1_md  # With metadata path
cshdd -a -n 11 -h /dev/sdb1 -t standard -o            # Read-only
cshdd -a -n 11 -h /dev/sdb1 -t standard -S            # Sync CRC
```

**Enable/Disable**:

```bash
cshdd -e -n 11 -h /dev/sdb1       # Enable
cshdd -e -n 11 -h /dev/sdb1 -D    # Enable without data restoration
cshdd -d -n 11 -h /dev/sdb1       # Disable
```

**Remove**:

```bash
cshdd -r -n 11 -h /dev/sdb1       # Remove
cshdd -r -n 11 -h /dev/sdb1 -f    # Force remove
```

**Modify/Enlarge**:

```bash
cshdd -m -n 11 -h /dev/sdb1 -t archive   # Modify type
cshdd -E -n 11 -h /dev/sdb1              # Enlarge
```

**Checksum Operations**:

```bash
cshdd -v -n 11 -h /dev/sdb1            # Verify checksums
cshdd -v -n 11 -h /dev/sdb1 -F         # Verify and fix
cshdd -v -n 11 -h /dev/sdb1 -A         # Abort on first mismatch
cshdd -R -n 11 -h /dev/sdb1            # Reset checksums
cshdd -C -n 11 -h /dev/sdb1            # Clear error flags
cshdd -I -n 11                          # Discard checksum cache
cshdd -y true -n 11 -h /dev/sdb1       # Enable checksum sync
```

## csbench — Benchmark Storage Performance

```bash
csbench -v 0 -s 10485760              # Basic 10MB benchmark
csbench -v 0 -s 1073741824 -b all     # 1GB sequential read/write
csbench -v 0 -s 536870912 -b random   # 512MB random I/O
csbench -v 0 -s 104857600 -C 4        # 100MB multi-client
csbench -v 0 -B -n 11 -h 2            # Background recovery benchmark
```

**Options**: `-v` volume, `-s` size bytes, `-p` passes, `-b` mode (all/read/write/random), `-C` clients, `-m/-M` min/max block size, `-S` sleep time.

## csioload — I/O Load Testing

```bash
csioload -v 0 -s scenario.json -r results.json
```

## csconf — Configure Storage Parameters

**Print Configuration**:

```bash
csconf -p                              # Show all defaults
```

**Modify by Node**:

```bash
csconf -m -n 11 -M 10737418240       # Max bg memory: 10GB
csconf -m -n 11 -S 5368709120        # Max other memory: 5GB
csconf -m -n 11 -o 8                  # Max concurrent bg ops
csconf -m -n 11 -b 1048576           # Max bytes per bg op
csconf -m -n 11 -g on                # Grouped I/O on
csconf -m -n 11 -s on                # I/O sorting on
csconf -m -n 11 -a on                # Async network on
```

**Object Store**:

```bash
csconf -m -n 11 -R 30000             # Request timeout (ms)
csconf -m -n 11 -T 10000             # Connect timeout (ms)
csconf -m -n 11 -X 4                  # Connection threads
csconf -m -n 11 -C 10                 # Connections
```

**Performance**:

```bash
csconf -m -n 11 -c 60                # I/O cleaner interval (s)
csconf -m -n 11 -P 1073741824        # Min network perf (B/s)
csconf -m -n 11 -w 90                # Space warning threshold (%)
csconf -m -n 11 -d 300               # Recovery delay (s)
```

## csdebug — Debug Storage Service

**Information**:

```bash
csdebug -p         # I/O queue info
csdebug -P         # All available info
csdebug -p -d      # Detailed I/O queue
```

**Consistency Checks**:

```bash
csdebug -c -n 11   # Check specific node
csdebug -c          # Check all nodes
```

**Force Sync**:

```bash
csdebug -S          # Force sync all EXAStorage nodes
```

**Debug Properties** (use with caution):

```bash
csdebug -s -t 5000              # Forced timeout
csdebug -s -D 10                # Network delay
csdebug -e 'CS_DEBUG_NWC=1'     # Set debug env vars
csdebug -s -a on                # Force STE_AGAIN
csdebug -s -r on                # Force metadata fail
```

## csrec — Storage Recovery Status

```bash
csrec -l                      # All volumes recovery status
csrec -l -v <volume-id>       # Specific volume
csrec -s -v <volume-id>       # Recovery percentage
csrec --check --volume=<id>   # Check integrity
csrec --repair --volume=<id>  # Repair volume
```

## csmd — Storage Metadata Operations

```bash
csmd -p -f <path-to-metadata-file>       # Print metadata file
csmd show --volume=<volume-id>            # Show volume metadata
csmd verify --volume=<volume-id>          # Verify integrity
csmd repair --volume=<volume-id>          # Repair metadata
```

**Metadata Locations**:

```
/exa/metadata/storage/metadata                    # Current
/exa/metadata/storage/metadata.csctrl.[timestamp] # Auto backups
/exa/metadata/storage/metadata.old                # Manual backup
```

**CRITICAL**: Never modify metadata files without backups. Corrupted metadata can cause complete data loss. Always backup before operations:

```bash
cp /exa/metadata/storage/metadata /root/metadata_backup_$(date +%Y%m%d_%H%M%S)
```

## csctrl — Storage Service Control

```bash
csctrl -s -c /exa/etc/cos_storage.conf                  # Start
csctrl --start --auto-add --auto-restart -n 11,12,13 -c /exa/etc/cos_storage.conf  # Start with options
csctrl -d                                                 # Stop
csctrl restart                                            # Restart
csctrl status                                             # Status
```

Always stop the database before stopping storage. Use `--auto-add` to auto-add available nodes.

## cscp — Copy Storage Data

```bash
cscp --source=<vol-id> --dest=<vol-id>          # Copy between volumes
cscp --source=<vol-id> --dest-node=<node-id>    # Copy to node
```

## csexport / csimport — Export and Import Data

```bash
csexport --volume=10001 --dest=s3://bucket/backup/ --compress
csimport --source=s3://bucket/backup/vol10001.tar.gz --volume=10001
```

## cos_log — Log to COS System

```bash
cos_log <service-name> <priority> <message>
```

Priorities: `emerg`, `alert`, `crit`, `err`, `warn`, `notice`, `info`, `debug`.

```bash
cos_log myservice info "Service started successfully"
cos_log myservice err "Connection failed"
```

## Storage Health Check Script

```bash
#!/bin/bash
echo "=== Volume Status ==="
csinfo -v

echo -e "\n=== HDD Status ==="
csinfo -H

echo -e "\n=== Recovery Status ==="
csrec -l

if csinfo -v | grep -q "OFFLINE"; then
    echo "[ALERT] Offline volumes detected!"
fi
```

## Metadata Restoration Procedure (Emergency Only)

```bash
# 1. Stop storage service
csctrl -d

# 2. Backup current metadata
cp /exa/metadata/storage/metadata /root/metadata_new_backup

# 3. Restore previous working metadata
cp /exa/metadata/storage/metadata.csctrl.[timestamp] /exa/metadata/storage/metadata

# 4. Start storage service
csctrl -s -c /exa/etc/cos_storage.conf

# 5. Verify HDDs are online
csinfo -H

# 6. Check device sizes
csinfo -D

# 7. Start database to verify
confd_client db_start db_name: Exasol
```

## Best Practices

- Create snapshots before any major changes: `cssnap -v <id> --name="before_$(date +%Y%m%d)"`
- Run weekly health checks: `csinfo -v && csinfo -D`
- Baseline performance monthly: `csbench -v 0 -s 1073741824 -b all`
- Always backup metadata before operations
- Label volumes clearly: `cslabel -a -v <id> -l "prod_data"`
- Monitor disk health: `csinfo -D | grep -E "FAILED|DEGRADED"`
- Stop database before stopping storage service
- Ensure sufficient free space before enlarging volumes

## Storage Troubleshooting

### Volume Not Accessible

```bash
csinfo -v | grep <volume-id>           # Check status
csinfo -V --id=<volume-id>             # Check nodes
csrec --check --volume=<volume-id>     # Check integrity
csrec --repair --volume=<volume-id>    # Repair
```

### Performance Degradation

```bash
csbench -t seq_read -s 1G --volume=<volume-id>   # Test performance
csinfo -D                                          # Check HDD health
```

### Disk Failure

```bash
csinfo -D | grep FAILED       # Identify failed disk
cshdd fail --id=<hdd-id>      # Mark as failed
cshdd add --device=/dev/sdX   # Add replacement
cshdd remove --id=<old-id>    # Remove old disk
```
