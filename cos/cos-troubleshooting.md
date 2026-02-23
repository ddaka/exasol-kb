---
tool_name: cos
doc_type: troubleshooting
category: COS Troubleshooting
title: "COS Troubleshooting"
summary: "cosps | grep dwad"
---
# COS Troubleshooting


## Common Issues

### Database Won't Start

#### Symptoms
- `confd_client -c db_start` completes but database not accessible
- Controller partition doesn't appear
- Controller exists but components missing

#### Diagnosis

**Check DWAD status**:
```bash
cosps | grep dwad
# Should show: ID 27, FLAGS: RA-I
```

**Check for controller**:
```bash
cosps | grep controller-Exasol
```

**Check components**:
```bash
DB_CONTROLLER=$(cosps | awk '/controller-Exasol/ {print $1}')
cosps -p $DB_CONTROLLER
# Should show 7 components
```

**Check logs**:
```bash
# ConfD logs
tail -100 /exa/logs/cored/confD.log | grep Exasol

# DWAD logs (if controller exists)
cosexec $DB_CONTROLLER -- tail -100 /exa/logs/db/Exasol/cored_Exasol_*.log
```

#### Solutions

**DWAD crashed**:
```bash
# DWAD auto-restarts, but need to start database manually
confd_client -c db_start -a "db: Exasol"
```

**Controller stuck**:
```bash
# Kill controller, let DWAD recreate
DB_CONTROLLER=$(cosps | awk '/controller-Exasol/ {print $1}')
coskill $DB_CONTROLLER

# Wait and start again
sleep 5
confd_client -c db_start -a "db: Exasol"
```

**Missing components**:
```bash
# Restart entire database
confd_client -c db_stop -a "db: Exasol"
sleep 5
confd_client -c db_start -a "db: Exasol"
```

**Corrupted metadata**:
```bash
# Check DWAD metadata
ls -la /exa/metadata/dwad/

# May need to restore from backup
# Contact Exasol support
```

---

### Partition Not Found

#### Symptoms
- `cosexec` returns "partition not found"
- Expected partition missing from `cosps` output
- Database stopped unexpectedly

#### Diagnosis

```bash
# List all partitions
cosps -f

# Check if partition ever existed (logs)
grep <partition-id> /exa/logs/logd/cos.log

# Check parent status
cosps | grep <parent-id>
```

#### Solutions

**Parent killed**:
- If parent terminated, all children are killed
- Restart parent service/database

**Manual kill**:
- Partition was manually killed
- Check if auto-restart flag (R) is set
- Restart service if needed

**Check for orphans**:
```bash
# Find partitions with missing parents
cosps | while read line; do
    ID=$(echo "$line" | awk '{print $1}')
    PARENT=$(echo "$line" | awk '{print $4}')
    if [ "$PARENT" != "0" ] && [ "$PARENT" != "PARENT" ]; then
        if ! cosps | grep -q "^$PARENT "; then
            echo "Orphan: $ID (parent $PARENT missing)"
        fi
    fi
done
```

---

### Connection Refused (SSH Port 20002)

#### Symptoms
- Cannot SSH to node on port 20002
- "Connection refused" or "Connection timed out"

#### Diagnosis

**Check sshd partition**:
```bash
cosps | grep sshd
# Should show: ID 16, FLAGS: R--I
```

**Check if port listening**:
```bash
# From node OS
netstat -tuln | grep 20002

# Or
ss -tuln | grep 20002
```

**Check from remote**:
```bash
telnet <node-ip> 20002
# Or
nc -zv <node-ip> 20002
```

#### Solutions

**sshd not running**:
```bash
# Check if crashed
cosps | grep sshd

# Should auto-restart (R flag)
# If not, check logs
tail -50 /exa/logs/logd/cos.log | grep sshd
```

**Firewall blocking**:
```bash
# Check firewall rules (from OS)
iptables -L -n | grep 20002

# Or
firewall-cmd --list-all | grep 20002
```

**Wrong port**:
- Verify using port **20002**, not 22
- SSH command: `ssh -p 20002 root@node-hostname`

---

### BucketFS Not Accessible

#### Symptoms
- UDFs fail to load
- Cannot access drivers
- HTTP 404 or connection refused on port 2580

#### Diagnosis

**Check BucketFS partition**:
```bash
cosps | grep bucketfs
# Should show: ID 25, OWNER: 500, FLAGS: RAEI
```

**Check files**:
```bash
cosexec 25 -- ls -la /exa/data/bucketfs/
```

**Check configuration**:
```bash
cat /exa/etc/bucketfs.cfg_bfsdefault
```

**Check HTTP access**:
```bash
curl http://localhost:2580/
# Should return bucket list or auth prompt
```

#### Solutions

**BucketFS crashed**:
```bash
# Auto-restarts with R flag
# Check logs
tail -50 /exa/logs/cored/bucketfs.log
```

**Permission issues**:
```bash
# Check ownership
ls -la /exa/data/bucketfs/

# Should be owned by exadefusr (500)
chown -R 500:500 /exa/data/bucketfs/
```

**Remote volumes misconfigured**:
```bash
# Check remote volume config
cat /exa/etc/remote_volumes/exadefusr.500.conf

# Verify S3/Azure credentials
```

**Restart BucketFS**:
```bash
dwacli bucketfs restart bfsdefault
```

---

### Database Components Missing

#### Symptoms
- `cosps -p <controller-id>` shows less than 7 components
- Database accessible but queries fail
- Specific operations don't work (e.g., IMPORT fails)

#### Diagnosis

**Count components**:
```bash
DB_CONTROLLER=$(cosps | awk '/controller-Exasol/ {print $1}')
COMPONENT_COUNT=$(cosps -p $DB_CONTROLLER | tail -n +2 | wc -l)
echo "Components: $COMPONENT_COUNT (expected: 7)"
```

**Check controller logs**:
```bash
cosexec $DB_CONTROLLER -- tail -100 /exa/logs/db/Exasol/cored_Exasol_*.log
```

**Identify missing**:
```bash
# Expected components:
# pddserver, objectserver, exasqllog, loaderd, exaetl, exacs, exasql
cosps -p $DB_CONTROLLER | awk '{print $8}'
```

#### Solutions

**Restart database**:
```bash
confd_client -c db_stop -a "db: Exasol"
sleep 5
confd_client -c db_start -a "db: Exasol"

# Verify all components started
cosps -p $(cosps | awk '/controller-Exasol/ {print $1}')
```

**Insufficient resources**:
- Check system RAM availability
- Check controller logs for OOM errors
- May need to reduce `-dbram` parameter

**Corrupted component**:
- Check individual component logs
- May need database recovery

---

### High Partition Count

#### Symptoms
- `cosps` shows hundreds of partitions
- System running slowly
- Orphaned database partitions

#### Diagnosis

```bash
# Count partitions
PARTITION_COUNT=$(cosps | tail -n +2 | wc -l)
echo "Total partitions: $PARTITION_COUNT"

# Find orphaned database partitions
cosps | grep -E "exasql|loader|pdd|object" | grep -v controller
```

#### Solutions

**Kill orphaned partitions**:
```bash
# Find and kill orphaned database components
cosps | grep -E "exasql|loader|pdd|object" | while read line; do
    ID=$(echo "$line" | awk '{print $1}')
    PARENT=$(echo "$line" | awk '{print $4}')
    
    # Check if parent exists
    if ! cosps | grep -q "^$PARENT "; then
        echo "Killing orphaned partition $ID"
        coskill $ID
    fi
done
```

**Restart services cleanly**:
```bash
# Stop all databases
confd_client -c db_stop -a "db: Exasol"

# Verify cleanup
cosps -p 27
# Should be empty or minimal
```

---

### Storage Issues

#### Symptoms
- Database reports I/O errors
- Queries fail with disk errors
- `csinfo -v` shows offline volumes

#### Diagnosis

**Check volumes**:
```bash
csinfo -v
# Look for OFFLINE or DEGRADED
```

**Check HDDs**:
```bash
csinfo -D
# Look for FAILED disks
```

**Check logs**:
```bash
tail -100 /exa/logs/cored/cos_storage.log
```

#### Solutions

**Volume offline**:
```bash
# Check and repair
csrec --check --volume=<volume-id>
csrec --repair --volume=<volume-id>
```

**Disk failed**:
```bash
# Mark as failed
cshdd fail --id=<hdd-id>

# Add replacement disk
cshdd add --device=/dev/sdX

# Remove failed disk
cshdd remove --id=<hdd-id>
```

**Restart storage service**:
```bash
# Check cos_storage partition
cosps | grep cos_storage

# Should auto-restart (RA-I flags)
# If not, check logs
```

---

### Node Offline

#### Symptoms
- `cosps -N` shows offline nodes
- `ONLINE NODES: 3/4` instead of `4/4`
- Database partially functional

#### Diagnosis

**Check node status**:
```bash
# Node information
coslookup -A --json

# Check specific node
coslookup -n <node-id> -v
```

**Check cluster consensus**:
```bash
coscheck
# Exit code 0 = healthy
```

**From offline node** (if accessible):
```bash
# Check COS services
cosps

# Check network
ping <other-node-ip>

# Check time sync
date
# Compare with other nodes
```

#### Solutions

**Network partition**:
- Check network connectivity between nodes
- Verify routing and firewall rules
- Check switch/network infrastructure

**Node crashed**:
- Check node OS logs
- Restart node if necessary
- Verify COS starts on boot

**Remove offline node**:
```bash
# If node permanently offline
cosrm -n <node-id> -f

# Reconfigure cluster for fewer nodes
```

---

## Debugging Workflows

### Database Startup Issues

```bash
#!/bin/bash
# Debug database startup

echo "=== Step 1: Check DWAD ==="
if ! cosps | grep -q "^27 "; then
    echo "❌ DWAD not running!"
    exit 1
fi
echo "✓ DWAD running"

echo -e "\n=== Step 2: Request database start ==="
echo "Starting database..."
confd_client -c db_start -a "db: Exasol"
sleep 10

echo -e "\n=== Step 3: Check for controller ==="
if ! DB_CONTROLLER=$(cosps | awk '/controller-Exasol/ {print $1}'); then
    echo "❌ Controller not created"
    echo "Check logs: tail /exa/logs/cored/confD.log"
    exit 1
fi
echo "✓ Controller exists: $DB_CONTROLLER"

echo -e "\n=== Step 4: Check components ==="
COMPONENT_COUNT=$(cosps -p $DB_CONTROLLER | tail -n +2 | wc -l)
echo "Components: $COMPONENT_COUNT / 7"

if [ $COMPONENT_COUNT -lt 7 ]; then
    echo "❌ Missing components"
    echo "Check logs: cosexec $DB_CONTROLLER -- tail /exa/logs/db/Exasol/cored_Exasol_*.log"
    exit 1
fi

echo -e "\n✓ Database started successfully"
```

---

### Performance Debugging

```bash
#!/bin/bash
# Debug performance issues

echo "=== Partition Resource Usage ==="
cosps -m | while read line; do
    ID=$(echo "$line" | awk '{print $1}')
    if [[ $ID =~ ^[0-9]+$ ]]; then
        echo "Partition $ID:"
        cosexec $ID -- ps aux | head -3
    fi
done

echo -e "\n=== Storage Performance ==="
csbench -t seq_read -s 1G

echo -e "\n=== Node Status ==="
coslookup -A --json | jq -r '.nodes[] | "\(.id)\t\(.state)"'

echo -e "\n=== Disk Health ==="
csinfo -D
```

---

## Collecting Diagnostic Information

### Full System Snapshot

```bash
#!/bin/bash
# Collect diagnostics for support

OUTPUT_DIR="/tmp/cos_diagnostics_$(date +%Y%m%d_%H%M%S)"
mkdir -p "$OUTPUT_DIR"

echo "Collecting diagnostics to $OUTPUT_DIR..."

# Partition information
cosps -f -m > "$OUTPUT_DIR/partitions.txt"
cospstree > "$OUTPUT_DIR/partition_tree.txt"

# Cluster information
cosinfo -e > "$OUTPUT_DIR/cluster_nodes.txt"
coslookup -A --json > "$OUTPUT_DIR/node_details.json"
coscheck > "$OUTPUT_DIR/health_check.txt" 2>&1

# Storage information
csinfo -v > "$OUTPUT_DIR/volumes.txt"
csinfo -D > "$OUTPUT_DIR/disks.txt"
csinfo -V > "$OUTPUT_DIR/volume_details.txt"

# Database state
confd_client -c info -j > "$OUTPUT_DIR/confd_info.json"
confd_client -c db_state -a "db: Exasol" -j > "$OUTPUT_DIR/db_state.json"

# Logs
cp -r /exa/logs/logd/*.log "$OUTPUT_DIR/" 2>/dev/null
cp -r /exa/logs/cored/*.log "$OUTPUT_DIR/" 2>/dev/null

# Create archive
tar czf "${OUTPUT_DIR}.tar.gz" "$OUTPUT_DIR"
echo "Diagnostics collected: ${OUTPUT_DIR}.tar.gz"
```

---

## Common Error Messages

### "Partition not found"
- Partition was killed or crashed
- Check parent partition status
- Review recent operations

### "Connection refused"
- Service not running (check with `cosps`)
- Wrong port (SSH uses 20002, not 22)
- Firewall blocking connection

### "No space left on device"
- Storage volumes full
- Check: `csinfo -v`
- Expand volumes or clean up data

### "Cannot start database"
- DWAD not running
- Corrupted metadata
- Insufficient resources
- Check DWAD and controller logs

### "Component failed to start"
- Resource constraints (RAM/CPU)
- Configuration error
- Check component-specific logs

---

## Getting Help

### Before Contacting Support

Collect this information:
1. Output of `cosps -f -m`
2. Output of `cospstree`
3. Output of `confd_client -c info -j`
4. Relevant logs from `/exa/logs/`
5. Description of issue and steps to reproduce
6. Recent changes or operations performed

### Exasol Support

- Support Portal: support.exasol.com
- Include diagnostic tarball
- Specify cluster ID: `cosinfo -c`
- Provide node hostnames and IDs

---

## Best Practices for Troubleshooting


**Check basics first**
- Is service running? (`cosps`)
- Are all nodes online? (`coslookup`)
- Any recent changes?


**Follow hierarchy**
- Check parent before children
- DWAD → Controller → Components


**Use logs effectively**
- Start with controller logs
- Check system logs in `/exa/logs/logd/`
- Look for timestamps matching issue


**Monitor while troubleshooting**
```bash
coswatch <partition-id> &
# See events in real-time
```


**Document steps taken**
- What was tried
- What worked/didn't work
- Helps avoid repeating failed attempts

## Related Documentation

- [COS Overview](cos_overview.md)
- [COS Partition Hierarchy](cos_partition_hierarchy.md)
- [COS System Partitions](cos_system_partitions.md)
- [COS Database Partitions](cos_database_partitions.md)
- [COS Commands Reference](cos_commands.md)
- [COS Storage Commands](cos_storage_commands.md)
- [COS Best Practices](cos_best_practices.md)
