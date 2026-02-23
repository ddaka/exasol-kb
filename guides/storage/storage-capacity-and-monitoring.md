---
tool_name: confd_client
doc_type: guide
category: storage
subcommands:
  - st_volume_enlarge
  - st_volume_info
  - st_volume_list
  - st_disk_list
  - st_disk_info
  - st_node_list
  - st_node_info
  - remote_volume_list
  - remote_volume_info
---

# Storage Capacity Management and Monitoring

This guide covers volume enlargement, usage monitoring, SQL-based capacity queries, and health check procedures for Exasol storage.

---

## Automatic Growth (Data Volumes)

**Data volumes grow automatically** as long as disk space is available.

**Triggers:**
- INSERT statements add data
- Tables grow
- Indexes expand

**No manual intervention needed** for normal growth.

---

## Manual Volume Enlargement

**When to enlarge manually:**
- Approaching maximum capacity
- Anticipating large data load
- Adding more disk space

**Procedure:**
```bash
# Enlarge data volume
confd_client st_volume_enlarge \
  vname: DataVolume1 \
  size: '2 TiB'

# Or by volume ID
confd_client st_volume_enlarge \
  vid: 2 \
  size: '2 TiB'
```

**For archive volumes:**
```bash
# Enlarge local archive volume
confd_client st_volume_enlarge \
  vname: LocalArchiveVolume1 \
  size: '3 TiB'
```

**Important:**
- Cannot shrink volumes
- Archive volumes do NOT grow automatically
- Remote archive volumes have no size limit (cloud storage)

---

## Check Volume Usage

### Via ConfD

```bash
# Get volume info
confd_client st_volume_info vname: DataVolume1

# List all volumes
confd_client st_volume_list
```

### Via SQL

```sql
-- Check volume sizes
SELECT * FROM EXA_VOLUME_SIZES;

-- Check database size
SELECT * FROM EXA_DB_SIZE_LAST_DAY;

-- Check by schema
SELECT * FROM EXA_SCHEMA_SIZES;

-- Check by table
SELECT * FROM EXA_ALL_OBJECT_SIZES
WHERE OBJECT_TYPE = 'TABLE'
ORDER BY MEM_OBJECT_SIZE DESC
LIMIT 20;
```

### Via File System

```bash
# Check disk usage
df -h /exa/data

# Check volume directory
du -sh /exa/data/storage/*
```

---

## Capacity Alerts

**Set up monitoring for:**
- Volume usage > 80%: Warning
- Volume usage > 90%: Critical
- Disk space < 10%: Critical

**SQL query for monitoring:**
```sql
SELECT 
  VOLUME_NAME,
  DEVICE_ID,
  (USED / SIZE) * 100 AS USAGE_PERCENT,
  CASE 
    WHEN (USED / SIZE) * 100 > 90 THEN 'CRITICAL'
    WHEN (USED / SIZE) * 100 > 80 THEN 'WARNING'
    ELSE 'OK'
  END AS STATUS
FROM EXA_VOLUME_SIZES
ORDER BY USAGE_PERCENT DESC;
```

---

## Key Metrics to Monitor

**Volume-level metrics:**
- Volume size (current and maximum)
- Volume usage percentage
- Free space remaining
- Growth rate (trend)
- Redundancy status

**Disk-level metrics:**
- Disk I/O (read/write throughput)
- Disk utilization (%)
- IOPS
- Latency
- Disk errors

**Backup-level metrics:**
- Backup size
- Backup duration
- Archive volume usage
- Expired backups
- Failed backups

---

## Monitoring Commands

### ConfD Commands

```bash
# Volume information
confd_client st_volume_list
confd_client st_volume_info vname: DataVolume1

# Disk information
confd_client st_disk_list
confd_client st_disk_info dname: disk1

# Storage nodes
confd_client st_node_list
confd_client st_node_info nid: 11

# Remote volumes
confd_client remote_volume_list
confd_client remote_volume_info remote_volume_name: S3Archive
```

### SQL Queries

```sql
-- Volume sizes and usage
SELECT * FROM EXA_VOLUME_SIZES ORDER BY DEVICE_ID;

-- Database size over time
SELECT * FROM EXA_DB_SIZE_LAST_DAY ORDER BY MEASURE_TIME DESC;

-- Storage-related system events
SELECT * FROM EXA_SYSTEM_EVENTS
WHERE DBMS_TEXT LIKE '%storage%'
  OR DBMS_TEXT LIKE '%volume%'
  OR DBMS_TEXT LIKE '%disk%'
ORDER BY MEASURE_TIME DESC
LIMIT 100;

-- Volume state
SELECT * FROM EXA_STATISTICS.EXA_MONITOR_LAST_DAY
WHERE MEASURE_NAME = 'VOLUME_STATE';
```

### System Commands

```bash
# Overall disk usage
df -h

# Specific directory usage
du -sh /exa/data/*

# Disk I/O statistics
iostat -x 5 10

# Check for disk errors in logs
grep -i "disk\|storage\|i/o error" /exa/logs/logd/Storage.log
```

---

## Volume Health Checks

### Daily Checks

```bash
# 1. Check volume status
confd_client st_volume_list

# 2. Check disk health
confd_client st_disk_list

# 3. Check for errors
tail -100 /exa/logs/logd/Storage.log | grep -i error
```

### Weekly Checks

```sql
-- Check growth trends
SELECT 
  INTERVAL_START,
  RAW_OBJECT_SIZE / 1024 / 1024 / 1024 AS SIZE_GB
FROM EXA_DB_SIZE_LAST_DAY
ORDER BY INTERVAL_START DESC
LIMIT 30;

-- Check fragmentation
SELECT * FROM EXA_STATISTICS.EXA_MONITOR_LAST_DAY
WHERE MEASURE_NAME LIKE '%FRAGMENTATION%';
```

## Related Documentation

- [Storage Overview](./storage-overview.md)
- [Storage Troubleshooting](./storage-troubleshooting.md)
- [Best Practices](./storage-best-practices.md)
