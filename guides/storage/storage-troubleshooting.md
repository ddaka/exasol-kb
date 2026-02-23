---
tool_name: confd_client
doc_type: troubleshooting
category: storage
subcommands:
  - st_volume_info
  - st_volume_enlarge
  - st_volume_create
  - st_disk_list
  - st_disk_info
  - remote_volume_info
  - remote_volume_modify
  - node_list
---

# Storage Troubleshooting

This document covers diagnosis and resolution of common Exasol storage problems: volume full, archive full, slow performance, disk failures, backup failures, and volume creation failures.

---

## Volume Full

**Symptom**: Writes fail with "volume full" or "insufficient space" errors

**Diagnosis:**
```bash
# Check volume usage
confd_client st_volume_info vname: DataVolume1

# Check disk space
df -h

# Check what's using space (SQL)
SELECT * FROM EXA_ALL_OBJECT_SIZES
ORDER BY MEM_OBJECT_SIZE DESC LIMIT 20;
```

**Solutions:**

1. **Enlarge volume:**
   ```bash
   confd_client st_volume_enlarge vname: DataVolume1 size: '2 TiB'
   ```

2. **Clean up data:**
   ```sql
   -- Drop unused tables
   DROP TABLE old_data;
   
   -- Truncate large tables
   TRUNCATE TABLE staging_data;
   
   -- Delete old records
   DELETE FROM logs WHERE log_date < ADD_DAYS(CURRENT_DATE, -90);
   ```

3. **Add more disks** (requires node reconfiguration)

---

## Archive Volume Full

**Symptom**: Backups fail with "archive volume full" error

**Diagnosis:**
```bash
confd_client st_volume_info vname: LocalArchiveVolume1
```

**Solutions:**

1. **Enable auto-cleanup:**
   ```bash
   # For remote volumes
   confd_client remote_volume_modify \
     remote_volume_name: S3Archive \
     options: 'cleanvolume'
   ```

2. **Manually delete old backups:**
   ```sql
   -- List backups
   SELECT * FROM EXA_DB_BACKUPS ORDER BY START_TIME;
   
   -- Delete specific backup
   DROP BACKUP 'backup_id';
   ```

3. **Enlarge local archive volume:**
   ```bash
   confd_client st_volume_enlarge \
     vname: LocalArchiveVolume1 \
     size: '4 TiB'
   ```

4. **Switch to remote archive** (unlimited capacity)

---

## Slow Storage Performance

**Symptom**: Queries slow, high I/O wait

**Diagnosis:**
```bash
# Check disk I/O
iostat -x 5

# Check for I/O errors
grep -i "i/o error" /exa/logs/logd/Storage.log

# Check load
top
```

**SQL diagnosis:**
```sql
-- Check for blocking operations
SELECT * FROM EXA_ALL_SESSIONS WHERE STATUS = 'EXECUTE';

-- Check query execution
SELECT * FROM EXA_SQL_LAST_DAY
WHERE DURATION > 60
ORDER BY DURATION DESC;
```

**Solutions:**

1. **Check disk health:**
   ```bash
   smartctl -a /dev/sdX
   ```

2. **Check for faulty disk:**
   ```bash
   confd_client st_disk_list
   # Look for degraded or failed status
   ```

3. **Optimize queries:**
   ```sql
   -- Add indexes
   CREATE INDEX idx_customer ON orders(customer_id);
   
   -- Update statistics
   ANALYZE TABLE orders;
   ```

4. **Check for contention:**
   - Too many concurrent writes
   - Large bulk loads

---

## Disk Failure

**Symptom**: Disk shows as "failed" or "degraded" in status

**Diagnosis:**
```bash
# Check disk status
confd_client st_disk_list
confd_client st_disk_info dname: disk1

# Check system events
grep -i "disk\|failed" /exa/logs/logd/Storage.log
grep -i "disk\|failed" /exa/logs/logd/eventd.log
```

**SQL diagnosis:**
```sql
SELECT * FROM EXA_SYSTEM_EVENTS
WHERE EVENT_TYPE = 'ERROR'
  AND DBMS_TEXT LIKE '%disk%'
ORDER BY MEASURE_TIME DESC;
```

**Solutions:**

1. **With redundancy ≥2**: System continues operating
   - Replace failed disk ASAP
   - System rebuilds data automatically

2. **Replace physical disk:**
   - Shutdown node
   - Replace hardware
   - Restart node
   - Rebuild process starts automatically

3. **Check RAID status** (if using RAID):
   ```bash
   cat /proc/mdstat
   ```

---

## Backup Failures

**Symptom**: BACKUP command fails

**Common errors:**
- "Cannot connect to remote volume"
- "Authentication failed"
- "Insufficient space"
- "Network timeout"

**Diagnosis:**
```bash
# Check remote volume config
confd_client remote_volume_info remote_volume_name: S3Archive

# Test connectivity
ping s3.eu-west-1.amazonaws.com

# Check logs
tail -100 /exa/logs/logd/ConfD.log | grep -i backup
```

**Solutions:**

1. **Authentication issues:**
   ```bash
   # Update credentials
   confd_client remote_volume_modify \
     remote_volume_name: S3Archive \
     username: NEW_KEY \
     password: NEW_SECRET
   ```

2. **Network issues:**
   - Check firewall rules
   - Check VPC endpoints (AWS)
   - Check service endpoints (Azure)
   - Test with verbose logging:
     ```bash
     confd_client remote_volume_modify \
       remote_volume_name: S3Archive \
       options: 'verbose'
     ```

3. **Timeout issues:**
   ```bash
   confd_client remote_volume_modify \
     remote_volume_name: S3Archive \
     options: 'timeout=1800'
   ```

---

## Volume Cannot Be Created

**Symptom**: `st_volume_create` fails

**Common errors:**
- "Insufficient disk space"
- "Invalid parameters"
- "Disk not found"

**Diagnosis:**
```bash
# Check disk availability
confd_client st_disk_list

# Check free space
df -h

# Check node availability
confd_client node_list
```

**Solutions:**

1. **Check disk name:**
   ```bash
   confd_client st_disk_list
   # Use exact disk name from output
   ```

2. **Check node IDs:**
   ```bash
   confd_client node_list
   # Ensure node IDs in volume create match active nodes
   ```

3. **Check parameters:**
   - `num_master_nodes` matches active node count
   - `redundancy` ≤ number of nodes
   - `size` doesn't exceed available disk space

## Related Documentation

- [Storage Capacity and Monitoring](./storage-capacity-and-monitoring.md)
- [Create Data Volume](./storage-create-data-volume.md)
- [Create Remote Archive Volume](./storage-create-remote-archive-volume.md)
- [Best Practices](./storage-best-practices.md)
