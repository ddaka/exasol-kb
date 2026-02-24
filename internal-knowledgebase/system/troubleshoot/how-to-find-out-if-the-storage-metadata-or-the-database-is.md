---
tool_name: cos
doc_type: troubleshoot
category: system
title: "How to find out if the storage metadata or the database is corrupt"
summary: "If a database does not start it can have multiple reasons:"
---
# How to find out if the storage metadata or the database is corrupt

## How to find out if the storage metadata or the database is corrupt

If a database does not start it can have multiple reasons:

1. Corrupt data

2. Not enough nodes online

3. Not all data segments online

4. Storage Metadata issues (corrupt, missing, wrong)

Number 2 and 3 can be answered easily - are all required volumes and nodes online? Even if a volume is degraded all data is available it only means that one segment has no redundancy segment available. Only locked volumes are missing data. Failed nodes can be replaced with spare nodes.

**Corrupt data** (1) happens very rarely and only if the underlying hardware or virtualisation platform has some kind of caching enabled or if hardware errors e.g. for hard disks are ignored or overseen for a very long time. How do I know that my data is corrupt?

### Backups are failing

Even though the network is fine and there is enough disk space available (doesn't matter if remote storage or inside the cluster SDFS). Each backup file has its own MD5sum, the MD5sum is generated while the file is written to the destination and then compared to the MD5sum of the file while verified (Backup stages: 5 % identify blocks to backup, 75% write backup to destination, 95% verify backup - read from destination, 100% cleanup expired files).

### The database won't start

Check the PDD Server log file on all nodes and search for "BLOCK MISSMATCH". The only way to fix this is a backup restore

### You cannot read specific blocks (data or any kind of objects) from the database

A specific query crashes while touching blocks on disk which cannot be read because they got corrupted (this will create Coredumps and also log entries in the PDD server). In that case do a backup restore or find out if the object can be removed from the database (btw. this might also lead to backups failing - of course only those touching these specific blocks...)

For corrupt data if might be worth checking if the metadata of the volume can be read and if the data is still integer.
Before running the next commands ensure the database is not running and the persistent data volume is unlocked and not in use.

```sql
[root@n0011 ~]# csinfo -v -i [VOL_ID] |grep -E "Curr|State"
State : ONLINE
Currently in use by : NOBODY
```
### Check the metadata of the database (execute on one of the DB nodes)

```bash
ssh n11
[root@n0011 ~]# cd /usr/opt/EXASuite-7/EXASolution-7.0.5/bin/
[root@n0011 bin]# ./checkdata -pers_volume_id 12 -mode print_metadata

print_metadata ok:
 node_id status start offset end_offset database_format node_count node_number tablespace   tan   write_time
 ------- ------ ------------ ---------- --------------- ---------- ----------- ---------- ----- ------------
       0     OK         8192 2147483648             000          4           0          0 25669 202105041523
       1     OK   2147491840 4294967296             000          4           1          0 25669 202105041523
       2     OK   4294975488 6442450944             000          4           2          0 25669 202105041523
       3     OK   6442459136 8589934592             000          4           3          0 25669 202105041523
```
If this command fails check the storage metadata. All nodes should have the same TAN and WRITE_TIME.

It is also possible to undo the last TAN but only do this with confirmation from R&D!

### Check the integrity of the database (execute on one of the DB nodes)

```bash
[root@n0011 bin]# ./checkdata -pers_volume_id 12 -mode data_integrity

data_integrity check ok:
 node_id status start offset end offset checked blocks phantom blocks missing blocks critical blocks
 ------- ------ ------------ ---------- -------------- -------------- -------------- ---------------
       0     OK         8192 2147483648            950              0              0               0
       1     OK   2147491840 4294967296            949              0              0               0
       2     OK   4294975488 6442450944            949              0              0               0
       3     OK   6442459136 8589934592            948              0              0               0
```
Status should be ok for all nodes, phantom, missing and critical blocks should be 0.

### . Corrupt storage metadata

If any of the above commands fails or gets stuck, then this is an indication that the metadata or hardware is corrupted.
