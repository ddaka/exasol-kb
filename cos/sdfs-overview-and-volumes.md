---
tool_name: sdfs
doc_type: concept
category: SDFS
title: "SDFS Overview and Volume Management"
summary: "SDFS (Secure Distributed File System) is Exasol's proprietary distributed file system. It provides virtual file storage across cluster nodes with automatic shar..."
---
# SDFS Overview and Volume Management

## Overview

SDFS (Secure Distributed File System) is Exasol's proprietary distributed file system. It provides virtual file storage across cluster nodes with automatic sharding, redundancy, and compression. SDFS is the storage layer for database data, backups, and BucketFS.

**Context**: All `sdfs` and volume management commands must be run **inside the COS namespace** (SSH port 20002).

## Key Characteristics

- Files are **virtual objects** in volumes, not regular Linux filesystem files
- Each volume has a numeric ID (local: `v0001`, remote: `r0001` or `10001+`)
- Block distribution (vertical/horizontal) for optimal performance
- Built-in compression for backup files
- FTP/SFTP access on port 2021 for external tools
- Compatible with remote storage (S3, Azure Blob, FTP, SMB, WebDAV)

## Volume Types

### Data Volumes (Local SDFS)

Store database data and temporary files. High-performance local storage.

Configured in EXAConf:

```ini
[EXAVolume : DataVolume1]
    Type = data
    Size = 8 GiB
    Disk = disk1
    Nodes = 11,12,13
    Redundancy = 2
```

### Local Archive Volumes

Store backups within the cluster using SDFS. Fast backup/restore, supports non-blocking and virtual restores.

```ini
[EXAVolume : archive]
    Type = archive
    Size = 10 GB
    Disk = disk1
    Nodes = 11
```

### Remote Archive Volumes

Store backups outside the cluster (S3, Azure Blob, FTP, SMB, WebDAV). Better disaster recovery (survives cluster failure), but slower than local. Only supports blocking restore.

## List Volumes

### List Local Volumes

```bash
confd_client st_volume_list
```

Output:

```yaml
- id: 0
  name: DataVolume1
  hdd_type: disk1
  nodes_list:
  - id: 11
    unrecovered_segments: 0
  redundancy: 1
  block_size: 4096
  owner: 500
  permissions: rwx------
```

### List Remote Volumes

```bash
confd_client remote_volume_list
```

Output:

```
- backup
- r0002
- nocompress
- r0004
```

### Get Remote Volume Details

```bash
confd_client remote_volume_list_details
```

Output:

```yaml
- name: backup
  type: s3
  url: https://10.70.0.171:9000/daka
  username: o3Xq1LgoWRE4a3sXdrOA
  password: eYFVaVNCNno2AZFjrUz1rdC4j6bpdT4JfyfwVwkG
  owner: [500, 500]
  vid: '10001'
  options: verbose, scheme=https, endpoint=https://10.70.0.171

- name: r0004
  type: webdav
  url: https://bucket.hcp.example.com/webdav/data/
  username: user1
  password: ExasolExasol123!
  owner: [500, 500]
  vid: '10004'
  options: webdav, noverifypeer, verbose
```

### Check Volume Configuration from EXAConf

```bash
grep -A10 "^\[EXAVolume" /exa/etc/EXAConf
```

## Create Remote Archive Volumes

### S3

```bash
confd_client remote_volume_add \
  vol_type: s3 \
  url: 's3://my-bucket/backups' \
  username: 'AWS_ACCESS_KEY' \
  password: 'AWS_SECRET_KEY' \
  owner: '[500,500]' \
  remote_volume_name: 's3_backup'
```

### FTP

```bash
confd_client remote_volume_add \
  vol_type: ftp \
  url: 'ftp://backup-server:2021/exasol/' \
  username: 'backup_user' \
  password: 'secure_password' \
  owner: '[500,500]'
```

### WebDAV

```bash
confd_client remote_volume_add \
  vol_type: webdav \
  url: 'https://webdav.server.com/backups/' \
  username: 'webdav_user' \
  password: 'password' \
  owner: '[500,500]' \
  options: '[noverifypeer, verbose]'
```

## SDFS Backup File Structure

Backups in SDFS follow this hierarchy:

```
DatabaseName/              # Database name (e.g., "Exasol")
  id_123/                  # Backup ID (unique identifier)
    level_0/               # Backup level: 0=full, 1+=incremental
      node_0/              # Node identifier
        backup_20260201    # Actual backup data
        metadata_20260201  # Backup metadata
      node_1/
        backup_20260201
    level_1/               # Incremental backup
      node_0/
        backup_20260202
        metadata_20260202
```

## Important Notes

- **Local volumes** (`v0001`, `v0002`) use SDFS storage within the cluster
- **Remote volumes** (`r0001`, `10001+`) point to external storage
- Volume IDs starting with `1000X` are remote volumes
- SDFS files are virtual — they don't exist as regular filesystem files
- Use `dwad_client list` to see which volumes databases are using
