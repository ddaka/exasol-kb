---
tool_name: confd_client
doc_type: guide
category: storage
subcommands:
  - st_volume_add_label
  - st_volume_enlarge
  - st_volume_create
  - remote_volume_add
---

# Exasol Storage Best Practices

This guide covers best practices for Exasol storage planning, volume management, backup strategy, performance optimization, capacity management, and security.

---

## Storage Planning

**1. Plan capacity with 50% overhead**
- Don't allocate 100% of disk space
- Leave room for temporary spikes
- Account for snapshots/backups

**2. Use LVM for flexibility**
- Easy resizing
- Persistent naming
- Simplified management

**3. Separate OS and data disks**
- OS: RAID 1, 200+ GB
- Data: Multiple disks, RAID optional
- Better performance and isolation

**4. Size archive volumes appropriately**
```
Archive size = Data size × 0.3 × Backup count × 1.2

Example:
- 1 TB data × 0.3 compression × 7 backups × 1.2 overhead = 2.5 TB archive
```

**5. Monitor storage trends**
- Track growth rate weekly
- Forecast capacity needs 3–6 months ahead
- Set up alerts at 80% usage

---

## Volume Management

**1. Use redundancy 2 for production**
- Standard for data volumes
- Standard for archive volumes
- Balances safety and capacity

**2. Enable volume optimizations**
```bash
confd_client st_volume_add_label vname: DataVolume1 label: useinitopt
confd_client st_volume_add_label vname: DataVolume1 label: usesearchopt
```

**3. Name volumes descriptively**
- `DataVolume1`, `DataVolume2`
- `LocalArchiveVolume1`
- `RemoteArchive_S3_Production`

**4. Document volume configuration**
- Keep record of volume IDs
- Document which database uses which volume
- Track changes in version control

---

## Backup Storage Strategy

**1. Use hybrid backup strategy**
- Local archive: Fast restores for recent backups
- Remote archive: Offsite protection for disaster recovery

**2. Remote archive best practices**
- Use cloud storage (S3/Azure/GCS) for unlimited capacity
- Enable `cleanvolume` option for automatic cleanup
- Use separate buckets/containers per cluster
- Enable versioning on cloud storage
- Set lifecycle policies for old backups

**3. Archive volume sizing**
```bash
# Local: 1-2 weeks of backups
LocalArchive = DataSize × 0.3 × 7 days × 1.2

# Remote: Long-term retention (30-90 days)
# Cloud storage scales automatically
```

**4. Test restores regularly**
- Monthly restore test from local archive
- Quarterly restore test from remote archive
- Measure restore times

---

## Performance Optimization

**1. Use fast local storage**
- NVMe SSD preferred
- SAS SSD acceptable
- Avoid spinning disks for data volumes

**2. Distribute I/O across multiple disks**
- Minimum 4 disks per node
- 8–12 disks for high-performance needs
- Balance across NUMA nodes (on NUMA systems)

**3. Monitor disk performance**
```bash
# Check I/O stats
iostat -x 5

# Look for:
# - %util < 80% (good)
# - await < 10ms (good)
# - High %util + high await = bottleneck
```

**4. Optimize backup timing**
- Schedule during off-peak hours
- Avoid concurrent heavy operations
- Use incremental backups when possible

---

## Capacity Management

**1. Set up automated monitoring**
```sql
-- Daily volume check
SELECT 
  VOLUME_NAME,
  ROUND((USED / SIZE) * 100, 2) AS USAGE_PCT,
  CASE 
    WHEN (USED / SIZE) * 100 > 90 THEN 'CRITICAL'
    WHEN (USED / SIZE) * 100 > 80 THEN 'WARNING'
    ELSE 'OK'
  END AS STATUS
FROM EXA_VOLUME_SIZES;
```

**2. Implement cleanup procedures**
- Drop unused tables
- Archive old data
- Compress historical tables
- Delete expired backups

**3. Plan for growth**
- Add capacity before reaching 80%
- Forecast based on historical trends
- Budget for storage expansion

**4. Consider data lifecycle**
- Frequently accessed: Fast storage
- Historical: Compressed tables
- Archived: External storage

---

## Security Considerations

**1. Encrypt sensitive volumes**
- Use LUKS for on-premise deployments
- Use cloud encryption (S3 SSE, Azure encryption)
- Encrypt archive volumes

**2. Secure backup credentials**
- Use IAM roles instead of keys (AWS)
- Rotate credentials regularly
- Use separate credentials per cluster

**3. Network security for remote archives**
- Use HTTPS/TLS for all connections
- Configure VPC endpoints (avoid internet)
- Whitelist cluster IP ranges

**4. Access control**
- Restrict volume ownership to database owner
- Use least-privilege IAM policies
- Audit access to backup storage

## Related Documentation

- [Storage Overview](./storage-overview.md)
- [Storage Requirements](./storage-requirements.md)
- [Storage Capacity and Monitoring](./storage-capacity-and-monitoring.md)
- [Storage Troubleshooting](./storage-troubleshooting.md)
