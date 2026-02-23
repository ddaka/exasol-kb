---
tool_name: confd_client
doc_type: guide
category: storage
subcommands:
  - st_disk_list
  - st_disk_info
  - st_node_list
---

# Exasol Storage Requirements

This guide describes general requirements and recommendations for storage in Exasol on-premise deployments, including supported technologies, minimum hardware specs, disk naming, sizing calculations, and performance considerations.

---

## Supported Storage Technologies

| Technology | Description | Use Case |
|------------|-------------|----------|
| **Sparse file devices** | Files on ext4/XFS filesystems | Development, testing |
| **Block devices** | Local SAS/SSD/NVMe | Production on-premise |
| **Virtual disks** | VM-attached storage | Virtual environments |
| **Remote storage** | iSCSI/SAN | Shared storage environments |
| **LVM2** | Logical Volume Manager | Recommended for flexibility |
| **LUKS** | Encrypted volumes | Security-sensitive deployments |

**Not supported:**
- NFS-mounted filesystems

---

## Minimum Requirements

**Storage drives:**
- **Minimum 4 drives** per node
- **Minimum 250 MBps** read/write capacity per drive
- **Recommended**: 8+ drives for better performance

**OS disks:**
- **RAID 1** or similar fault tolerance
- **150 GiB free** after installation
- Separate from data disks

**Swap partition:**
- Size per OS vendor recommendation (typically 16–32 GiB)

---

## Disk Naming Requirements

**Critical**: Disk labels like `/dev/nvme1n1` are **NOT persistent** and must **NOT** be used.

**Recommended approach**: Use **LVM2** for persistent block device names.

**Alternative**: Use explicit paths from `/dev/disk/by-id/`:
```bash
# Good: Persistent by-id path
/dev/disk/by-id/nvme-Amazon_EC2_NVMe_Instance_Storage_AWS1234567890

# Bad: Non-persistent device name
/dev/nvme1n1
```

**Why LVM2 is recommended:**
- Persistent device naming
- Flexible capacity management
- Easy resizing and migration
- Industry standard

---

## Sizing Calculations

### Data Volume Sizing

```
Required disk space = Raw data size × Compression factor × Redundancy level × Safety margin

Where:
- Compression factor: ~0.3 (70% compression typical)
- Redundancy level: 2 (for k=2 redundancy)
- Safety margin: 1.2-1.5 (20-50% overhead)

Example:
- Raw data: 1 TB
- Compressed: 1 TB × 0.3 = 300 GB
- With redundancy 2: 300 GB × 2 = 600 GB
- With 50% margin: 600 GB × 1.5 = 900 GB required
```

### Archive Volume Sizing

```
Archive size = Data volume size × Compression ratio × Number of backups × Retention policy

Example:
- Data volume: 1 TB
- Compressed backup: ~200-300 GB per backup
- 7 daily backups: ~1.4-2.1 TB
- 4 weekly backups: +800 GB - 1.2 TB
- Total: ~2.2-3.3 TB archive volume
```

For detailed sizing calculations, see Exasol Sizing Guidelines.

---

## Performance Considerations

**Disk performance targets:**
- **IOPS**: 10,000+ per node (combined)
- **Throughput**: 1+ GB/s per node (combined)
- **Latency**: <10ms average

**Recommended hardware:**
- **Local SSD/NVMe**: Best performance
- **High-speed SAN**: Acceptable if low latency
- **Network storage**: Only if meets performance requirements

## Related Documentation

- [Storage Overview](./storage-overview.md)
- [Disk Management](./storage-disk-management.md)
- [Best Practices](./storage-best-practices.md)
