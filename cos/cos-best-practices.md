---
tool_name: cos
doc_type: guide
category: COS Best Practices
title: "COS Best Practices"
summary: "cosps -m | grep -v \"1/1\"  # Find offline partitions"
---
# COS Best Practices


## Operational Best Practices

### Monitoring Partitions


**Regular health checks**
```bash
# Daily partition status
cosps -m | grep -v "1/1"  # Find offline partitions

# Database components
DB_CONTROLLER=$(cosps | awk '/controller/ {print $1}')
cosps -p $DB_CONTROLLER | wc -l  # Should be 8 (7 + header)
```


**Watch critical services**
```bash
# DWAD (all databases depend on it)
cosps | grep "^27 "

# BucketFS (UDFs and drivers)
cosps | grep bucketfsd

# ConfD (configuration management)
cosps | grep confd
```


**Set up monitoring alerts**
- Partition offline/crashed
- Missing database components
- DWAD restarts
- Storage volume offline
- Node failures

---

### Database Lifecycle Management


**Always use proper tools**
```bash
# Correct way
confd_client db_start db: Exasol
confd_client db_stop db: Exasol

# Wrong way
coskill 56832  # Don't do this
```


**Graceful shutdown sequence**
```bash
# 1. Stop accepting connections
# 2. Complete in-flight transactions
# 3. Stop database via confd_client
confd_client db_stop db: Exasol

# 4. Verify stopped
cosps -p 27  # Should be empty or minimal

# 5. Then stop nodes if needed
```


**Verify component count**
```bash
# After database start, always check
DB_CONTROLLER=$(cosps | awk '/controller-Exasol/ {print $1}')
if [ -n "$DB_CONTROLLER" ]; then
    COMPONENTS=$(cosps -p $DB_CONTROLLER | tail -n +2 | wc -l)
    if [ $COMPONENTS -eq 7 ]; then
        echo "[OK] All components running"
    else
        echo "[WARN] Only $COMPONENTS/7 components running"
    fi
fi
```

---

### Partition Management


**Never kill child partitions directly**
```bash
# Avoid: killing child partitions directly
coskill 56840  # exasql component

# Preferred: work through parent
confd_client db_restart db: Exasol

# Or kill controller (DWAD manages restart)
coskill 56832  # Controller will handle children
```


**Understand flag implications**
- **R flag**: Auto-restarts (safe to kill)
- **No R flag**: Manual restart needed (don't kill)
- **E flag**: External access (clients connect here)
- **K flag**: Persistent (partition remains after exit)


**Check before killing**
```bash
# See what you're about to kill
cosps -f | grep <partition-id>

# See what children will be affected
cospstree <partition-id>
```

---

### Storage Management


**Regular storage health checks**
```bash
#!/bin/bash
# Weekly storage check

echo "=== Volumes ==="
csinfo -v | grep -E "OFFLINE|DEGRADED" && echo "[WARN] Issues found" || echo "[OK] All volumes online"

echo -e "\n=== Disks ==="
csinfo -D | grep -E "FAILED|DEGRADED" && echo "[WARN] Issues found" || echo "[OK] All disks healthy"
```


**Snapshot before major operations**
```bash
# Before updates
cssnap -v <volume-id> --name="pre_update_$(date +%Y%m%d)"

# Before schema changes
cssnap -v <volume-id> --name="pre_migration_$(date +%Y%m%d)"

# Cleanup old snapshots
cssnap --list | grep "pre_update" | head -n -5 | while read snap; do
    cssnap --delete --id=$(echo $snap | awk '{print $1}')
done
```


**Monitor storage performance**
```bash
# Monthly baseline
csbench --full -s 5G > /var/log/storage_baseline_$(date +%Y%m).txt

# Compare to previous baselines
# Alert on significant degradation (>20%)
```

---

### Security Best Practices


**Secure SSH access**
- Use key-based authentication
- Disable password authentication
- Limit SSH access to specific IPs
- Use jump hosts for production


**Partition isolation**
```bash
# Check partition ownership
cosps -m | awk '{print $2,$8}' | sort | uniq

# Database should run as exadefusr (500)
# System services as root
```


**Audit partition access**
```bash
# Log all cosexec commands
alias cosexec='echo "$(date): cosexec $*" >> /var/log/partition_access.log; cosexec'

# Review periodically
tail -50 /var/log/partition_access.log
```

---

### Backup and Recovery


**Regular backups**
```bash
# Automated daily backups
confd_client db_backup_add_schedule \
    db: Exasol, enabled: true, hour: 2, minute: 0, level: 0

# Weekly storage snapshots
cssnap -v <volume-id> --name="weekly_$(date +%Y_W%V)"
```


**Test restore procedures**
```bash
# Monthly restore test
# 1. Create test database
# 2. Restore from backup
# 3. Verify data integrity
# 4. Document results
```


**Backup retention**
- Production: 30 days minimum
- Development: 7 days
- Test: 3 days
- Off-site copy: 90 days

---

### Multi-Node Cluster Operations


**Check node distribution**
```bash
# Ensure partitions span nodes properly
cosps -N

# Check for unbalanced distribution
cosps | awk '{print $6}' | sort | uniq -c
```


**Graceful node maintenance**
```bash
# 1. Drain node (move partitions)
for pid in $(cosps -n <node-id> | awk '{print $1}' | tail -n +2); do
    cosmv -N <target-node> $pid
done

# 2. Verify node empty
cosps -n <node-id>

# 3. Perform maintenance

# 4. Re-add node
cosadd -N <node-name> -x
```


**Monitor cluster health**
```bash
# Regular cluster checks
coscheck || echo "[WARN] Cluster health issues detected"

# Node status
coslookup -A --json | jq -r '.nodes[] | select(.state!="online") | "\(.id): \(.state)"'
```

---

### Performance Optimization


**Minimize cosexec overhead**
```bash
# Inefficient - multiple executions
cosexec 56832 -- ls /exa
cosexec 56832 -- pwd
cosexec 56832 -- id

# Efficient - single execution
cosexec 56832 -- bash -c 'ls /exa && pwd && id'
```


**Use appropriate monitoring frequency**
```bash
# Real-time (for debugging only)
coswatch 56832

# Regular (for monitoring)
*/5 * * * * cosps | grep -v "1/1" && alert  # Every 5 min

# Periodic (for reporting)
0 0 * * * cosps -f > /var/log/daily_partition_status.log  # Daily
```


**Storage performance**
```bash
# Pre-allocate volumes for known growth
csvol create --size=<future-size> --type=data

# Use appropriate snapshot retention
# Don't keep too many snapshots (impacts performance)
```

---

### Change Management


**Document all changes**
```bash
# Maintain change log
echo "$(date): Database restarted for patch" >> /var/log/cos_changes.log
echo "$(date): Node 12 added to cluster" >> /var/log/cos_changes.log
```


**Test in non-production first**
```bash
# Development → Staging → Production
# Verify each step before proceeding
```


**Have rollback plan**
```bash
# Before change
cssnap -v <volume-id> --name="pre_change_$(date +%Y%m%d_%H%M%S)"

# If issues, rollback
cssnap --restore --id=<snapshot-id>
```

---

### Automation Best Practices


**Health check script**
```bash
#!/bin/bash
# /usr/local/bin/cos_health_check.sh

ALERT_EMAIL="ops@example.com"
ISSUES=""

# Check DWAD
if ! cosps | grep -q "^27 "; then
    ISSUES="$ISSUES\nDWAD not running!"
fi

# Check database components
DB_CONTROLLER=$(cosps | awk '/controller-Exasol/ {print $1}')
if [ -n "$DB_CONTROLLER" ]; then
    COMPONENT_COUNT=$(cosps -p $DB_CONTROLLER | tail -n +2 | wc -l)
    if [ $COMPONENT_COUNT -lt 7 ]; then
        ISSUES="$ISSUES\nDatabase has only $COMPONENT_COUNT/7 components"
    fi
fi

# Check storage
if csinfo -v | grep -q "OFFLINE"; then
    ISSUES="$ISSUES\nOffline storage volumes detected"
fi

# Send alert if issues
if [ -n "$ISSUES" ]; then
    echo -e "COS Health Issues:\n$ISSUES" | mail -s "COS Alert" $ALERT_EMAIL
    exit 1
fi

exit 0
```

**Schedule health checks**:
```bash
# Crontab
*/5 * * * * /usr/local/bin/cos_health_check.sh
```

---

### Logging and Auditing


**Centralize logs**
```bash
# Forward COS logs to central logging
# rsyslog, fluentd, or similar

# Key logs to collect:
# - /exa/logs/logd/*.log
# - /exa/logs/cored/*.log
# - /exa/logs/db/*/
```


**Retention policies**
```bash
# System logs: 30 days
find /exa/logs/logd/ -name "*.log" -mtime +30 -delete

# Database logs: 90 days
find /exa/logs/db/ -name "*.log" -mtime +90 -delete

# Core dumps: 7 days (after analysis)
find /exa/logs/cored/ -name "core.*" -mtime +7 -delete
```


**Audit trail**
```bash
# Log all administrative actions
echo "$(date) $(whoami): Started database" >> /var/log/cos_audit.log
```

---

### Disaster Recovery


**Regular DR drills**
```bash
# Quarterly test:
# 1. Simulate node failure
# 2. Verify failover
# 3. Test recovery procedures
# 4. Document results
```


**Multi-region backups**
```bash
# Export backups to different region
csexport --volume=<volume-id> --dest=s3://dr-bucket/backups/

# Verify backup integrity
csimport --source=s3://dr-bucket/backups/latest --verify
```


**Recovery Time Objective (RTO)**
- Document acceptable downtime
- Test recovery procedures
- Automate where possible

---

### Documentation Standards


**Maintain runbooks**
- Database startup/shutdown procedures
- Common troubleshooting steps
- Escalation paths
- Contact information


**Document cluster configuration**
```bash
# Generate cluster documentation
cat > /docs/cluster_config.md << EOF
# Exasol Cluster Configuration

## Node Information
$(coslookup -A --json | jq)

## Partitions
$(cosps -f -m)

## Storage
$(csinfo -v)

## Last Updated: $(date)
EOF
```


**Keep change log**
- All configuration changes
- Version upgrades
- Incident resolutions
- Lessons learned

---

## Quick Reference Checklist

### Daily Operations
- [ ] Check partition status (`cosps | grep -v "1/1"`)
- [ ] Verify database running (`cosps -p 27`)
- [ ] Review logs for errors
- [ ] Check storage health (`csinfo -v`)

### Weekly Operations
- [ ] Storage health check (`csinfo -D`)
- [ ] Review backup completion
- [ ] Clean up old logs
- [ ] Check cluster health (`coscheck`)

### Monthly Operations
- [ ] Test database restore from backup
- [ ] Performance baseline (`csbench`)
- [ ] Review and clean old snapshots
- [ ] Update documentation

### Quarterly Operations
- [ ] DR drill
- [ ] Security audit
- [ ] Capacity planning review
- [ ] Update runbooks

---

## Common Mistakes to Avoid


** Don't kill database components directly**
```bash
# Wrong
coskill 56840  # exasql component

# Right
confd_client db_restart db: Exasol
```


** Don't ignore offline partitions**
```bash
# Check regularly
cosps | grep -v "1/1"  # Find issues
```


** Don't skip backups before changes**
```bash
# Always backup first
cssnap -v <volume-id> --name="pre_change"
```


** Don't restart DWAD unless necessary**
```bash
# DWAD restart = all databases stop
# Use database-specific tools instead
```


** Don't ignore storage warnings**
```bash
# Act on DEGRADED/OFFLINE volumes immediately
csinfo -v
```

---

## Emergency Procedures

### Database Down

```bash
# 1. Check DWAD
cosps | grep dwad

# 2. Check logs
tail -100 /exa/logs/cored/confD.log

# 3. Restart database
confd_client db_start db: Exasol

# 4. Verify components
cosps -p $(cosps | awk '/controller/ {print $1}')
```

---

### Node Failure

```bash
# 1. Identify failed node
coslookup -A --json

# 2. Check impact
cosps -N | grep "<failed-node>"

# 3. If permanent, remove node
cosrm -n <node-id> -f

# 4. Rebalance if needed
```

---

### Storage Failure

```bash
# 1. Identify failed volume/disk
csinfo -v
csinfo -D

# 2. Mark disk as failed
cshdd fail --id=<hdd-id>

# 3. Add replacement
cshdd add --device=/dev/sdX

# 4. Remove failed disk
cshdd remove --id=<hdd-id>
```

---

## Related Documentation

- [COS Overview](cos_overview.md)
- [COS Partition Hierarchy](cos_partition_hierarchy.md)
- [COS System Partitions](cos_system_partitions.md)
- [COS Database Partitions](cos_database_partitions.md)
- [COS Commands Reference](cos_commands.md)
- [COS Storage Commands](cos_storage_commands.md)
- [COS Troubleshooting](cos_troubleshooting.md)
