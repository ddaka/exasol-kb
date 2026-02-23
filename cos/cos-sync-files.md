---
tool_name: cos
doc_type: reference
category: COS File Operations
title: "cos_sync_files — File Synchronization Across Cluster Nodes"
summary: "`cos_sync_files` synchronizes files or directories from the current node to all other nodes in the Exasol cluster. Use it to deploy configuration files, scripts..."
---
# cos_sync_files — File Synchronization Across Cluster Nodes

## Overview

`cos_sync_files` synchronizes files or directories from the current node to all other nodes in the Exasol cluster. Use it to deploy configuration files, scripts, binaries, or certificates cluster-wide.

**Location**: `/opt/exasol/cos-<version>/sbin/cos_sync_files`

**Context**: Must be executed **inside the COS namespace** (SSH port 20002), not on the host machine.

## Prerequisites

- SSH access to Exasol COS namespace: `ssh root@<host> -p 20002`
- Exasol COS v8.0 or later
- Multi-node cluster (single-node clusters show "No nodes to synchronize")

## Basic Syntax

```bash
cos_sync_files <file_or_directory_path>
```

Files are copied to the **same path** on all target nodes. Existing files are **overwritten without prompting**.

## Examples

### Sync a Single File

```bash
echo "production settings" > /tmp/config.txt
cos_sync_files /tmp/config.txt

# Verify on all nodes
psh "cat /tmp/config.txt"
```

### Sync a Directory

```bash
mkdir -p /opt/exasol/scripts
cp monitoring.sh /opt/exasol/scripts/
cos_sync_files /opt/exasol/scripts/

# Verify
psh "ls -l /opt/exasol/scripts/"
```

### Sync Configuration File

```bash
vi /etc/telegraf/telegraf.conf
cos_sync_files /etc/telegraf/telegraf.conf

# Verify file contents match via hash
psh "md5sum /etc/telegraf/telegraf.conf"
```

All MD5 hashes should be identical across nodes.

## Common Workflows

### Deploy Monitoring Agent Configuration

```bash
vi /etc/telegraf/telegraf.conf
cos_sync_files /etc/telegraf/telegraf.conf
psh "pkill -HUP telegraf"
psh "ps aux | grep telegraf | grep -v grep"
```

### Update SSL Certificates

```bash
cp new_cert.pem /exa/etc/ssl/
cos_sync_files /exa/etc/ssl/new_cert.pem
psh "openssl x509 -in /exa/etc/ssl/new_cert.pem -noout -dates"
```

### Distribute Custom Scripts

```bash
cat > /opt/exasol/check_health.sh << 'EOF'
#!/bin/bash
df -h / | tail -1
free -h | grep Mem
EOF

chmod +x /opt/exasol/check_health.sh
cos_sync_files /opt/exasol/check_health.sh
psh "/opt/exasol/check_health.sh"
```

## cos_sync_files vs SDFS

These serve different purposes:

- **cos_sync_files**: Syncs regular filesystem files between cluster nodes.
- **SDFS operations** (`sdfs addraw`/`sdfs getraw`): Manage virtual files in SDFS volumes (backup storage).

```bash
# Sync regular file to all nodes
cos_sync_files /etc/config.txt

# Upload file into SDFS volume (different purpose)
sdfs addraw v0001 /etc/config.txt Exasol/config.txt
```

## Important Notes

- Source file must exist on the node where you run `cos_sync_files`
- Use **absolute paths** for reliability
- Permissions and ownership are **preserved** during synchronization
- Directories are synced **recursively**
- Always **verify after sync** using `psh`

## Troubleshooting

### "No nodes to synchronize"

Normal in single-node setups. In multi-node clusters:
- Check node states: `confd_client node_list`
- Verify network connectivity between nodes

### Permission Denied

- Ensure read access to the source file
- Run `cos_sync_files` as root
- Check target directory permissions: `psh "ls -ld /target/directory"`

### Files Not Appearing After Sync

- Verify source file exists: `ls -l /path/to/file`
- Check target nodes: `psh "ls -l /path/to/file"`
- Check disk space: `psh "df -h"`
