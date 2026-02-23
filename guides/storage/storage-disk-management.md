---
tool_name: confd_client
doc_type: reference
category: storage
subcommands:
  - st_disk_list
  - st_disk_info
  - st_node_list
  - st_node_info
---

# Exasol Disk Management

This document covers disk configuration, setup via c4, LVM configuration, and commands for checking disk status in Exasol on-premise deployments.

---

## Disk Configuration

**Disks** are the underlying physical or virtual storage devices that volumes use.

**Typical configuration:**
```
Node n11:
  - OS disk: /dev/sda (RAID 1, 200 GB)
  - Data disk 1: /dev/disk/by-id/nvme-disk1 (1 TB NVMe)
  - Data disk 2: /dev/disk/by-id/nvme-disk2 (1 TB NVMe)
  - Data disk 3: /dev/disk/by-id/nvme-disk3 (1 TB NVMe)
  - Data disk 4: /dev/disk/by-id/nvme-disk4 (1 TB NVMe)
```

---

## Disk Setup in c4

**During deployment**, disks are specified in the c4 configuration:

```bash
# config file
CCC_HOST_DATADISK="/dev/nvme1n1,/dev/nvme2n1,/dev/nvme3n1,/dev/nvme4n1"
```

**With LVM (recommended):**
```bash
# Create LVM volumes first
pvcreate /dev/nvme1n1
vgcreate exadata /dev/nvme1n1
lvcreate -l 100%FREE -n data1 exadata

# Then reference in config
CCC_HOST_DATADISK="/dev/exadata/data1,/dev/exadata/data2,..."
```

---

## Check Disk Status

### Via ConfD (confd_client)

```bash
# List all storage nodes
confd_client st_node_list

# Get disk information
confd_client st_disk_list

# Check disk health
confd_client st_disk_info dname: disk1

# Get node details
confd_client st_node_info nid: 11
```

### Via File System

```bash
# Check disk space
df -h

# Check physical disks
lsblk

# Check LVM status
lvdisplay
pvdisplay
vgdisplay

# Check disk I/O
iostat -x 5
```

### Via SQL

```sql
-- Check storage volumes
SELECT * FROM EXA_VOLUME_SIZES;

-- Check disk usage
SELECT * FROM EXA_DB_SIZE_LAST_DAY;

-- Check storage errors
SELECT * FROM EXA_SYSTEM_EVENTS 
WHERE EVENT_TYPE = 'ERROR' 
  AND DBMS_TEXT LIKE '%storage%';
```

## Related Documentation

- [Storage Requirements](./storage-requirements.md)
- [Create Data Volume](./storage-create-data-volume.md)
- [Storage Troubleshooting](./storage-troubleshooting.md)
