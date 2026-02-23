---
tool_name: c4
doc_type: guide
category: c4 Removing
title: "c4 Removing Deployments"
summary: "Permanently remove an Exasol deployment and all associated resources."
---
# c4 Removing Deployments

## Remove a Deployment

Permanently remove an Exasol deployment and all associated resources.

## ⚠️ CRITICAL WARNING

**This action is IRREVERSIBLE!**

`c4 rm` will:
- Delete all EC2 instances
- Delete all EBS volumes
- **Permanently destroy all data**
- Cannot be undone

**Create backups before removing deployments!**

## Syntax

```bash
c4 rm [-P|-p PLAY_ID] [-i CONFIG] [--yes]
```

## Parameters

| Parameter | Description |
|-----------|-------------|
| **-P** | Remove current deployment |
| **-p PLAY_ID** | Remove specified deployment |
| **-i CONFIG** | **(Jump host only)** Path to config file used for deployment<br>Omit when running from database host |
| **--yes** | Skip confirmation prompt |

## Examples

### Remove from Jump Host

```bash
c4 rm -i ./config -p f7fdff8e
```

### Remove from Database Host

```bash
c4 rm -p f7fdff8e
```

### Remove with Auto-Confirmation

```bash
c4 rm -i ./config -p f7fdff8e --yes
```

### Remove Current Deployment

```bash
c4 rm -P
```

## Pre-Removal Checklist

Before removing a deployment:

1. **Create final backup**:
   ```bash
   c4 connect -i PLAY_ID -s cos -- 'confd_client db_backup_start db_name: MY_DATABASE'
   ```

2. **Download critical files from BucketFS**:
   - UDF scripts
   - Configuration files
   - Custom packages

3. **Export configurations**:
   - Database settings
   - Network configurations
   - Access credentials

4. **Notify stakeholders**:
   - Users affected by shutdown
   - Dependent services
   - Operations team

5. **Update documentation**:
   - Remove from inventory
   - Update network diagrams
   - Archive configurations

## Safe Removal Procedure

### Step 1: Stop Database

```bash
c4 connect -i PLAY_ID -s cos -- 'confd_client db_stop db_name: MY_DATABASE'
```

### Step 2: Verify Database Stopped

```bash
c4 connect -i PLAY_ID -s cos -- 'confd_client db_state db_name: MY_DATABASE'
```

### Step 3: Stop Nodes

```bash
c4 down PLAY_ID
```

### Step 4: Remove Deployment

```bash
c4 rm -i ./config -p PLAY_ID
```

### Step 5: Verify Removal

Check AWS console:
- EC2 instances terminated
- EBS volumes deleted
- Security groups removed (if unique)
- No lingering resources

## What Gets Deleted

### AWS Resources Removed

| Resource Type | Action |
|---------------|--------|
| **EC2 Instances** | Terminated |
| **EBS Volumes** | Deleted (data lost) |
| **Elastic IPs** | Released (if allocated by c4) |
| **Security Groups** | Removed (if unique to deployment) |
| **Network Interfaces** | Detached and deleted |

### What Remains

| Resource Type | Status |
|---------------|--------|
| **VPC** | Not deleted (shared resource) |
| **Subnets** | Not deleted (shared resource) |
| **Route Tables** | Not deleted (shared resource) |
| **IAM Roles** | Not deleted (shared resource) |
| **CloudWatch Logs** | May remain (retention policy) |
| **S3 Backups** | Not deleted (manual cleanup needed) |

## Post-Removal Cleanup

### Manual Cleanup Required

1. **S3 Backups**:
   - Review S3 bucket
   - Delete old backups if not needed
   - Manage retention policy

2. **CloudWatch Logs**:
   - Review log groups
   - Delete old logs
   - Update retention

3. **IAM Credentials**:
   - Revoke access keys
   - Remove users specific to deployment
   - Update policies

## Troubleshooting

### Removal Fails

**Check**:
- AWS permissions sufficient
- No dependent resources blocking deletion
- Network connectivity to AWS

**Solution**:
```bash
# Retry with verbose output
c4 rm -i ./config -p PLAY_ID -v
```

### Orphaned Resources

If removal incomplete:

1. **Check AWS Console**:
   - Look for running instances with PLAY_ID tag
   - Check for attached EBS volumes
   - Review security groups

2. **Manual cleanup**:
   - Terminate instances manually
   - Delete volumes manually
   - Remove security groups if safe

## Alternative: Stop Instead of Remove

If you may need the deployment again:

**Stop nodes instead**:
```bash
# Stop database first
c4 connect -i PLAY_ID -s cos -- 'confd_client db_stop db_name: MY_DATABASE'

# Then stop nodes
c4 down PLAY_ID
```

**Benefits**:
- Data preserved
- Can restart later
- No AWS charges for stopped instances (only storage)

## Best Practices

**Always create final backup**
- Full database backup
- BucketFS files
- Configuration files

**Document removal reasons**
- Audit trail
- Compliance requirements
- Future reference

**Verify complete removal**
- Check AWS console
- Confirm no orphaned resources
- Update inventory

**Clean up manually-created resources**
- S3 backups
- CloudWatch logs
- IAM credentials

**Update documentation**
- Remove from systems inventory
- Archive configurations
- Update network diagrams

## Related Documentation

- [c4 Overview](c4_overview.md)
- [c4 Managing Nodes](c4_managing_nodes.md)
- [c4 Creating Deployments](c4_creating_deployments.md)
- [Backup and Restore](https://docs.exasol.com/db/latest/administration/on-premise/backup_restore.htm)
