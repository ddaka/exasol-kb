---
tool_name: confd_client
doc_type: reference
category: storage
subcommands:
  - remote_volume_add
  - remote_volume_list
  - remote_volume_info
  - remote_volume_delete
  - remote_volume_modify
  - db_list
  - db_info
---

# Create Remote Archive Volume

This document explains how to create remote archive volumes for database backups stored on external services outside the Exasol cluster. Covers all supported platforms: Amazon S3, Azure Blob Storage, Google Cloud Storage, FTP/FTPS, SFTP, Samba (SMB), WebDAV, and Apache Hadoop (WebHDFS).

**Advantages:**
- No cluster storage used
- Offsite backup capability
- Unlimited capacity (pay-as-you-go)
- Multiple cloud availability zones

**Considerations:**
- Network bandwidth required
- Egress costs (cloud providers)
- Slower restore than local
- External service dependencies

---

## Prerequisites

### General Prerequisites

- All nodes must reach the remote target
- Read-write access on remote host
- Network connectivity from cluster to remote service

### Amazon S3 Prerequisites

- S3 bucket URL: `https://<bucket>.s3.<region>.amazonaws.com/[optional-dir/]`
- AWS access key ID and secret access key
- Read-write permissions on bucket
- If nodes on private network, configure S3 VPC endpoint and update route table

### Azure Blob Storage Prerequisites

- Container URL: `https://<storage>.blob.core.windows.net/<container>`
- Storage account access key
- Read-write permissions
- If private network, configure service endpoint for Microsoft.Storage

### Google Cloud Storage Prerequisites

- Bucket URL: `https://<bucket>.storage.googleapis.com`
- Access key for Cloud Storage account
- Read-write permissions
- If private network, enable Private Google Access in subnet

### FTP/SFTP Prerequisites

- FTP server URL: `ftp[s]://<server>:<port>/optional-dir/`
- Username and password
- Read-write access
- Supports multiple servers (comma or range notation), custom ports, TLS/SSL

**Example URLs:**
```bash
# Single server
ftps://backup.example.com:2021/exasol/

# Multiple servers (range)
ftp://backup1..backup5.example.com/exasol/

# Multiple servers (specific)
ftp://10.0.0.1,10.0.0.3,10.0.0.6:2021/backups/
```

---

## Procedure

### Step 1: Connect to COS

```bash
c4 connect -i <PLAY_ID> -s cos
```

### Step 2: Find Database Name and Owner

```bash
# List databases
confd_client db_list

# Get database owner
confd_client db_info db_name: MY_DATABASE | grep owner -A 2
```

**Example output:**
```
owner:
- 500
- 500
```

### Step 3: Create Remote Archive Volume

**General syntax:**
```bash
confd_client remote_volume_add \
  url: <URL> \
  vol_type: <TYPE> \
  remote_volume_name: <NAME> \
  username: <USERNAME> \
  password: <PASSWORD> \
  owner: [<UID>, <GID>] \
  [options: '<OPTION1>,<OPTION2>,...']
```

### Parameters

| Parameter | Type | Description | Required |
|-----------|------|-------------|----------|
| `url` | string | URL of remote storage | Yes |
| `vol_type` | string | Type: s3, azure, gs, ftp, sftp, smb, webhdfs, webdav, file | Yes |
| `remote_volume_name` | string | Name for the volume | No (auto-generated) |
| `username` | string | Username/access key for remote service | For most types |
| `password` | string | Password/secret key for remote service | For most types |
| `owner` | tuple/list | Database owner [UID, GID] or [user, group] | Yes |
| `remote_volume_id` | integer | Volume ID (must be ≥10000, 5 digits) | No (auto-generated) |
| `options` | list | Additional options (comma-separated) | No |
| `labels` | list | Volume labels | No |

---

## Examples by Platform

### Amazon S3

```bash
confd_client remote_volume_add \
  url: https://my-bucket.s3.eu-west-1.amazonaws.com/cluster1/ \
  vol_type: s3 \
  remote_volume_name: RemoteArchiveVolume_S3 \
  username: AKIAIOSFODNN7EXAMPLE \
  password: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY \
  owner: '[500, 500]'
```

### Azure Blob Storage

```bash
confd_client remote_volume_add \
  url: https://exasolbackups.blob.core.windows.net/cluster1 \
  vol_type: azure \
  remote_volume_name: RemoteArchiveVolume_Azure \
  username: exasolbackups \
  password: <AZURE_STORAGE_ACCESS_KEY> \
  owner: '[500, 500]'
```

### Google Cloud Storage

```bash
confd_client remote_volume_add \
  url: https://exasol-backups.storage.googleapis.com \
  vol_type: gs \
  remote_volume_name: RemoteArchiveVolume_GCS \
  username: <GCS_ACCESS_KEY_ID> \
  password: <GCS_SECRET_ACCESS_KEY> \
  owner: '[500, 500]'
```

### FTP/FTPS

```bash
confd_client remote_volume_add \
  url: ftps://backup.example.com:2021/exasol/ \
  vol_type: ftp \
  remote_volume_name: RemoteArchiveVolume_FTP \
  username: backup_user \
  password: backup_password \
  owner: '[500, 500]' \
  options: 'forcessl'
```

### SFTP

```bash
confd_client remote_volume_add \
  url: sftp://backup.example.com:2022/backups/ \
  vol_type: sftp \
  remote_volume_name: RemoteArchiveVolume_SFTP \
  username: exasol \
  password: <SFTP_PASSWORD> \
  owner: '[500, 500]'
```

### Samba (SMB)

```bash
confd_client remote_volume_add \
  url: smb://fileserver.example.com:2139/exasol-backups/ \
  vol_type: smb \
  remote_volume_name: RemoteArchiveVolume_SMB \
  username: DOMAIN\\backup_user \
  password: <SMB_PASSWORD> \
  owner: '[500, 500]'
```

### Apache Hadoop (WebHDFS)

```bash
confd_client remote_volume_add \
  url: https://hadoop-namenode.example.com:20443/exasol/ \
  vol_type: webhdfs \
  remote_volume_name: RemoteArchiveVolume_Hadoop \
  username: hdfs \
  password: <HDFS_TOKEN> \
  owner: '[500, 500]' \
  options: 'webhdfs'
```

### WebDAV

```bash
confd_client remote_volume_add \
  url: https://webdav.example.com/backups/ \
  vol_type: webdav \
  remote_volume_name: RemoteArchiveVolume_WebDAV \
  username: backup_user \
  password: <WEBDAV_PASSWORD> \
  owner: '[500, 500]' \
  options: 'webdav'
```

---

## Volume Options

Add options with `options: 'option1,option2,...'`:

| Option | Description | Applies To |
|--------|-------------|------------|
| `cleanvolume` | Auto-remove expired backups after successful backup | All |
| `nocompression` | Store raw (uncompressed) backups | All |
| `noverifypeer` | Disable SSL certificate verification | All |
| `forcessl` | Use STARTTLS | FTP |
| `earlytls` | Allow TLS < 1.2 | All |
| `noresume` | Disable retry/resume on errors | All |
| `verbose` | Detailed logging (for troubleshooting) | All |
| `timeout=<sec>` | Override default timeout (e.g., timeout=300) | All |
| `proxy=<proxy>` | Use proxy server | All |
| `user=<key>` | Alternative way to set S3 access key | S3 |
| `webhdfs` | Enable WebHDFS protocol | WebHDFS |
| `webdav` | Enable WebDAV for HTTP URLs | WebDAV |
| `delegation_token=<token>` | Hadoop delegation token | WebHDFS |

**Example with options:**
```bash
confd_client remote_volume_add \
  url: https://my-bucket.s3.eu-west-1.amazonaws.com/ \
  vol_type: s3 \
  remote_volume_name: S3Archive \
  username: AKIAIOSFODNN7EXAMPLE \
  password: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY \
  owner: '[500, 500]' \
  options: 'cleanvolume,timeout=600,verbose'
```

---

## Verification

```bash
# List remote volumes
confd_client remote_volume_list

# Check specific volume
confd_client remote_volume_info remote_volume_name: RemoteArchiveVolume_S3
```

**Example output:**
```
name: RemoteArchiveVolume_S3
owner:
- 500
- 500
password: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
type: s3
url: https://my-bucket.s3.eu-west-1.amazonaws.com/
username: AKIAIOSFODNN7EXAMPLE
vid: '10001'
```

---

## Using Remote Archive Volumes

**In BACKUP command:**
```sql
-- S3
BACKUP DATABASE MY_DATABASE TO REMOTE ARCHIVE 'RemoteArchiveVolume_S3';

-- Azure
BACKUP DATABASE MY_DATABASE TO REMOTE ARCHIVE 'RemoteArchiveVolume_Azure';

-- FTP
BACKUP DATABASE MY_DATABASE TO REMOTE ARCHIVE 'RemoteArchiveVolume_FTP';
```

---

## Remove Remote Archive Volume

```bash
confd_client remote_volume_delete remote_volume_name: RemoteArchiveVolume_S3

# Or by ID
confd_client remote_volume_delete remote_volume_id: 10001
```

## Related Documentation

- [Storage Overview](./storage-overview.md)
- [Create Local Archive Volume](./storage-create-local-archive-volume.md)
- [Storage Troubleshooting](./storage-troubleshooting.md)
