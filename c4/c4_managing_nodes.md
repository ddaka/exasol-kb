---
tool_name: c4
doc_type: guide
category: c4 Node Management
title: "c4 Managing Nodes"
summary: "Start all nodes or specific nodes in a deployment."
---
# c4 Managing Nodes

## Start Nodes

Start all nodes or specific nodes in a deployment.

### Start All Nodes

```bash
c4 up PLAY_ID
```

### Start Specific Nodes

```bash
c4 up --nodes NODE_ID PLAY_ID
```

### Examples

```bash
# Start all nodes in deployment f7fdff8e
c4 up f7fdff8e

# Start only node 12 in deployment f7fdff8e
c4 up --nodes 12 f7fdff8e

# Start multiple specific nodes
c4 up --nodes 11,12,13 f7fdff8e
```

### What `c4 up` Does

- Equivalent to starting EC2 instances in AWS console
- Starts the nodes (instances) but **does not start the database**
- Use this when the access node is offline

**To start the database**, see [Start a Database](https://docs.exasol.com/db/latest/administration/aws/manage_database/start_db.htm).

## Stop Nodes

Stop all nodes or specific nodes in a deployment.

### Stop All Nodes

```bash
c4 down PLAY_ID
```

### Stop Specific Nodes

```bash
c4 down --nodes NODE_ID PLAY_ID
```

### Examples

```bash
# Stop all nodes in deployment f7fdff8e
c4 down f7fdff8e

# Stop only node 12 in deployment f7fdff8e
c4 down --nodes 12 f7fdff8e
```

## ⚠️ CRITICAL WARNING

`c4 down` does **NOT** safely shut down the database and may leave it in a corrupted state!

### What `c4 down` Does

- Equivalent to stopping EC2 instances in AWS console
- Forces node shutdown without graceful database shutdown
- **Risk of data loss and corruption**

## Proper Shutdown Procedure

**Always stop the database before using `c4 down`**

### Step-by-Step Shutdown

1. **Stop the database**:
   ```bash
   c4 connect -i PLAY_ID -s cos -- 'confd_client db_stop db_name: MY_DATABASE'
   ```

2. **Verify database stopped**:
   ```bash
   c4 connect -i PLAY_ID -s cos -- 'confd_client db_state db_name: MY_DATABASE'
   ```

3. **Then stop nodes**:
   ```bash
   c4 down PLAY_ID
   ```

## Recovery from Improper Shutdown

If you already used `c4 down` without stopping the database:

1. **Start nodes**: 
   ```bash
   c4 up PLAY_ID
   ```

2. **Restore from backup**
   - See [Backup and Restore](https://docs.exasol.com/db/latest/administration/on-premise/backup_restore.htm)

3. **Check database state**:
   ```bash
   c4 connect -i PLAY_ID -s cos -- 'confd_client db_state db_name: MY_DATABASE'
   ```

## Best Practices

**Always stop database before stopping nodes**
- Prevents data corruption
- Use `db_stop` ConfD job before `c4 down`

**Create backups before major operations**
- Before updates
- Before removing deployments
- Before scaling operations

**Verify node state**
- Use `c4 ps` to check node status
- Ensure all operations complete successfully

## Related Documentation

- [c4 Overview](c4_overview.md)
- [c4 Connecting to Deployments](c4_connecting.md)
- [c4 Monitoring Deployments](c4_monitoring.md)
- [Stop a Database](https://docs.exasol.com/db/latest/administration/aws/manage_database/stop_db.htm)
- [Stop a Cluster](https://docs.exasol.com/db/latest/administration/aws/manage_clusters/stop_cluster.htm)
- [Start a Database](https://docs.exasol.com/db/latest/administration/aws/manage_database/start_db.htm)
