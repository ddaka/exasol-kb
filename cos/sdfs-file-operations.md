---
tool_name: sdfs
doc_type: reference
category: SDFS
title: "SDFS File Operations: Download, Upload, and List Files"
summary: "This guide covers downloading, uploading, and listing files in SDFS (Secure Distributed File System) volumes. SDFS files are virtual objects — you must use `sdf..."
---
# SDFS File Operations: Download, Upload, and List Files

## Overview

This guide covers downloading, uploading, and listing files in SDFS (Secure Distributed File System) volumes. SDFS files are virtual objects — you must use `sdfs` commands or FTP to access them.

**Context**: All `sdfs` commands must be run **inside the COS namespace** (SSH port 20002).

**Key Commands**: `sdfs getraw` (download), `sdfs addraw` (upload), `sdfs list` (detailed listing), `sdfs shortlist` (compact listing)

## Quick Reference

```bash
# Download from SDFS
sdfs getraw v0001 path/to/file > local_file

# Upload to SDFS
sdfs addraw v0001 /local/file path/in/sdfs

# List files (compact)
sdfs shortlist v0001

# List files (detailed with sizes)
sdfs list v0001
```

## Download Files from SDFS

### From Local Volume

```bash
# Download file from local volume v0001
sdfs getraw v0001 Exasol/backups/backup_file.dat > /tmp/backup_file.dat

# Download backup data
sdfs getraw v0001 Database/id_123/level_0/node_0/backup_20260201 > backup.tar
```

### From Remote Volume

```bash
# Download from remote volume (S3, WebDAV, FTP, etc.)
sdfs getraw r0004 Exasol/data/myfile.txt > /tmp/myfile.txt

# Download from remote volume by numeric ID
sdfs getraw 10001 backups/archive.tar.gz > /tmp/archive.tar.gz
```

### Via FTP/SFTP (From Outside the Cluster)

SDFS exposes volumes via FTP on port 2021:

```bash
# Using curl
curl -u admin:password \
  ftp://10.70.0.171:2021/v0001/Exasol/backups/file.dat \
  -o local_file.dat

# Using wget (recursive download)
wget --user=admin --password=password \
  --recursive --level=5 \
  --directory-prefix=/tmp/backups \
  ftp://10.70.0.171:2021/v0001/Database/id_123/

# Using sftp
sftp -P 2021 admin@10.70.0.171:/v0001/Exasol/backups/file.dat
```

### Pipe to Another Command

```bash
# Download and decompress
sdfs getraw v0001 backup.gz | gunzip > backup.tar

# Transfer between volumes
sdfs getraw v0001 file.dat | sdfs addraw v0002 - file.dat

# Download from remote cluster via SSH
ssh -c aes128-ctr root@remote_node \
  'sdfs getraw r0001 DB/id_123/level_0/node_0/backup_file' > /tmp/backup_file
```

## Upload Files to SDFS

### From Local File

```bash
# Upload to local volume
sdfs addraw v0001 /tmp/myfile.txt Exasol/data/myfile.txt

# Upload to remote volume
sdfs addraw r0004 /path/to/backup.tar Database/id_123/level_0/node_0/backup_20260201
```

### From stdin (Pipe)

```bash
# Pipe data directly into SDFS
cat myfile.dat | sdfs addraw v0001 - Exasol/backups/myfile.dat

# Compress and upload
gzip -c backup.tar | sdfs addraw v0001 - backups/backup.tar.gz

# Upload from remote source
curl https://example.com/file.zip | sdfs addraw v0001 - downloads/file.zip
```

### Via FTP (From Outside the Cluster)

```bash
# Upload using curl (creates directories automatically)
curl --ftp-create-dirs -T backup_file.dat \
  -u admin:password \
  ftp://10.70.0.171:2021/v0001/Exasol/backups/

# Upload multiple files
for file in *.dat; do
  curl --ftp-create-dirs -T "$file" \
    -u admin:password \
    ftp://10.70.0.171:2021/v0001/Exasol/backups/
done
```

## List Files in SDFS Volumes

### Short Format

```bash
sdfs shortlist v0001
```

Output:

```
Exasol/id_123/level_0/node_0/backup_20260201
Exasol/id_123/level_0/node_0/metadata_20260201
Exasol/id_123/level_0/node_1/backup_20260201
```

### Detailed Format

```bash
sdfs list v0001
```

Output:

```
       SIZE         USAGE         ARCHIVED         MODIFIED           EXPIRE          DELETED   NAME
      57.9851       57.9854 1970-01-01 01:00 1970-01-01 01:00                -                -   backup.tar
       0.0000        0.0000 1970-01-01 01:00 1970-01-01 01:00                -                -   Exasol/

TOTAL:  SIZE N/A   USED 7422.1250   FREE N/A  TRASH N/A   (all sizes in MiB)
```

### Filter Listings

```bash
# Filter by database name
sdfs shortlist v0001 | grep "^Exasol/"

# Find specific backup
sdfs shortlist v0001 | grep "id_123/level_0"

# List specific path in detailed format
sdfs list v0001 Exasol/backups/
```

## Common Use Cases

### Download a Specific Backup

```bash
# List available backups
sdfs shortlist v0001 | grep "Exasol/id_"

# Download backup files
sdfs getraw v0001 Exasol/id_123/level_0/node_0/backup_20260201 > backup_node0.dat
sdfs getraw v0001 Exasol/id_123/level_0/node_0/metadata_20260201 > metadata_node0.dat
```

### Copy Backups Between Volumes

```bash
# Copy from remote to local volume
sdfs getraw r0001 Exasol/id_123/level_0/node_0/backup_file | \
  sdfs addraw v0001 - Exasol/id_123/level_0/node_0/backup_file

# Copy between clusters via SSH
ssh -c aes128-ctr root@cluster-a 'sdfs getraw v0001 path/to/file' | \
  sdfs addraw v0001 - path/to/file
```

### Verify File Integrity

```bash
# Calculate MD5 checksum of SDFS file
sdfs getraw v0001 Exasol/file.dat | md5sum

# Compare files between volumes
diff <(sdfs getraw v0001 file.dat) <(sdfs getraw r0004 file.dat)
```

## Troubleshooting

### Cannot Access Remote Volume

`sdfs getraw r0004 file` fails with connection error.

- Check credentials: `confd_client remote_volume_list_details`
- Test network connectivity from COS namespace
- Verify remote storage service is accessible
- Check options (e.g., `noverifypeer` for SSL issues)

### File Not Found in Volume

- List volume contents: `sdfs shortlist v0001`
- Check exact path (case-sensitive)
- Verify correct volume ID
- Ensure file hasn't been deleted

### Permission Denied on Upload

- Check volume ownership: `confd_client st_volume_list`
- Run as root or volume owner (typically UID 500)
- Verify volume permissions

### FTP Access Not Working

- Verify SDFS FTP service is running in COS namespace
- Check firewall rules for port 2021
- Test credentials
- Ensure volume is accessible (not offline)

## Important Notes

- **Volume IDs**: Local = `v0001`, Remote = `r0001` or `10001+`
- **FTP access** is available on port 2021 for external tools
- **Compression** is automatic for remote archive volumes (unless disabled)
- **Use pipes** for efficient data transfer without temporary files
- SDFS files are virtual — standard Linux file commands (`ls`, `cat`, `cp`) do not work on them
