---
tool_name: c4
doc_type: troubleshooting
category: c4 Troubleshooting
title: "c4 Troubleshooting"
summary: "Error: Operation timed out after 30m"
---
# c4 Troubleshooting

## Common Issues and Solutions

### Timeout Errors

#### Problem
```
Error: Operation timed out after 30m
```

#### Cause
- Operation took longer than timeout setting
- Network issues
- AWS resource provisioning delays

#### Solution
```bash
# Increase timeout and retry
CCC_PLAY_TIMEOUT=1h c4 apply -P -i ./config
```

See [c4 Timeout Configuration](c4_timeouts.md) for details.

---

### Authentication Errors

#### Problem
```
Error: Unable to locate credentials
```

#### Cause
- AWS credentials not configured
- Credentials expired
- MFA token required

#### Solution

**Check credentials**:
```bash
aws sts get-caller-identity
```

**Configure credentials**:
```bash
aws configure
```

**With MFA**:
```bash
aws sts get-session-token \
  --serial-number arn:aws:iam::123456789012:mfa/username \
  --token-code 123456
```

See [c4 MFA Documentation](c4_mfa.md) for details.

---

### Permission Errors

#### Problem
```
Error: User is not authorized to perform: ec2:RunInstances
```

#### Cause
- IAM permissions insufficient
- Wrong AWS account/region
- Resource quotas exceeded

#### Solution

**Check IAM policy**:
```bash
aws iam get-user-policy --user-name USERNAME --policy-name POLICY
```

**Required permissions**:
- `ec2:*`
- `iam:PassRole`
- `sts:GetSessionToken` (if using MFA)

**Contact AWS administrator** to grant permissions.

---

### Config File Errors

#### Problem
```
Error: Invalid configuration file
```

#### Cause
- Syntax error in config file
- Missing required parameters
- Wrong file path

#### Solution

**Verify config file**:
```bash
cat ./config
```

**Check syntax**:
- YAML/JSON format correct
- No trailing commas
- Quotes properly closed

**Validate required parameters**:
- `region`
- `availability_zone`
- `node_size`
- `cluster_size`

See [c4 Configuration](c4_configuration.md) for examples.

---

### Network Errors

#### Problem
```
Error: Unable to establish SSH connection
```

#### Cause
- Security group rules blocking SSH
- Network ACLs blocking traffic
- Wrong SSH key
- Instance not running

#### Solution

**Check instance status**:
```bash
c4 ps -p PLAY_ID
```

**Verify security groups**:
```bash
aws ec2 describe-security-groups --group-ids SG_ID
```

**Test SSH manually**:
```bash
ssh -i ~/.ssh/exasol.pem admin@INSTANCE_IP
```

**Check network ACLs** in AWS console.

---

### Deployment Not Found

#### Problem
```
Error: Deployment PLAY_ID not found
```

#### Cause
- Wrong PLAY_ID
- Deployment in different region
- Deployment already removed

#### Solution

**List all deployments**:
```bash
c4 ls
```

**Check correct region**:
```bash
aws configure get region
```

**Search by tag**:
```bash
aws ec2 describe-instances --filters "Name=tag:Name,Values=*exasol*"
```

---

### Database Won't Start

#### Problem
```
Error: Database failed to start
```

#### Cause
- Corrupted database
- Insufficient resources
- License issues
- Nodes not properly shutdown

#### Solution

**Check database state**:
```bash
c4 connect -i PLAY_ID -s cos -- 'confd_client db_state db_name: MY_DATABASE'
```

**Check logs**:
```bash
c4 connect -i PLAY_ID -s cos -- 'cat /exa/logs/cored/*'
```

**Restore from backup**:
```bash
c4 connect -i PLAY_ID -s cos -- 'confd_client db_backup_list'
```

See [Start a Database](https://docs.exasol.com/db/latest/administration/aws/manage_database/start_db.htm).

---

### Update Failures

#### Problem
```
Error: Cluster update failed
```

#### Cause
- Database not stopped
- Wrong version number
- Network issues
- Insufficient disk space

#### Solution

**Stop database first**:
```bash
c4 connect -i PLAY_ID -s cos -- 'confd_client db_stop db_name: MY_DATABASE'
```

**Verify database stopped**:
```bash
c4 connect -i PLAY_ID -s cos -- 'confd_client db_state db_name: MY_DATABASE'
```

**Retry update**:
```bash
CCC_PLAY_TIMEOUT=1h c4 update cluster -p PLAY_ID -t @exasol-2025.1.0
```

See [c4 Updating Deployments](c4_updating.md).

---

### c4 down Corrupted Database

#### Problem
Database corrupted after using `c4 down` without stopping database first.

#### Cause
- Used `c4 down` without `db_stop`
- Database not properly shutdown

#### Solution

**Start nodes**:
```bash
c4 up PLAY_ID
```

**Restore from backup**:
```bash
c4 connect -i PLAY_ID -s cos -- 'confd_client db_backup_list'
c4 connect -i PLAY_ID -s cos -- 'confd_client db_restore backup_id: BACKUP_ID'
```

**Prevention**: Always stop database before stopping nodes!

See [c4 Managing Nodes](c4_managing_nodes.md).

---

### Resource Quota Exceeded

#### Problem
```
Error: You have exceeded your limit for instances in this region
```

#### Cause
- AWS account limits reached
- Too many existing instances

#### Solution

**Check current usage**:
```bash
aws ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId,State.Name]' --output table
```

**Check quotas**:
```bash
aws service-quotas list-service-quotas --service-code ec2 --query 'Quotas[?QuotaName==`Running On-Demand Standard instances`]'
```

**Request quota increase**:
- AWS Console → Service Quotas
- Request increase for "Running On-Demand Instances"

**Temporary**: Remove unused deployments
```bash
c4 ls  # Find unused deployments
c4 rm -p OLD_PLAY_ID
```

---

### Disk Space Issues

#### Problem
```
Error: No space left on device
```

#### Cause
- EBS volumes full
- Database logs consuming space
- BucketFS files too large

#### Solution

**Check disk usage**:
```bash
c4 connect -i PLAY_ID -s cos -- 'df -h'
```

**Clean up logs**:
```bash
c4 connect -i PLAY_ID -s cos -- 'find /exa/logs -name "*.log" -mtime +30 -delete'
```

**Expand EBS volume**:
1. AWS Console → EC2 → Volumes
2. Modify volume size
3. Resize filesystem

---

### Connection Refused

#### Problem
```
Error: Connection refused when connecting to deployment
```

#### Cause
- Nodes not running
- Service not started
- Firewall blocking connection
- Wrong hostname/port

#### Solution

**Check node status**:
```bash
c4 ps -p PLAY_ID
```

**Start nodes if stopped**:
```bash
c4 up PLAY_ID
```

**Test connectivity**:
```bash
# Test from jump host
telnet INSTANCE_IP 8563

# Check service status
c4 connect -i PLAY_ID -s cos -- 'systemctl status cored'
```

---

### Slow Performance

#### Problem
Operations taking very long to complete.

#### Cause
- Network latency
- AWS region overloaded
- Instance type undersized
- Disk I/O bottleneck

#### Solution

**Check network**:
```bash
ping -c 10 ec2.us-east-1.amazonaws.com
```

**Monitor AWS status**:
- https://status.aws.amazon.com/

**Consider larger instances**:
- Update config with larger `node_size`
- Recreate deployment

**Check disk I/O**:
```bash
c4 connect -i PLAY_ID -s cos -- 'iostat -x 5 3'
```

---

## Debugging Commands

### Check Deployment Status

```bash
c4 ps -p PLAY_ID
```

### View Logs

```bash
# ConfD logs
c4 connect -i PLAY_ID -s cos -- 'cat /exa/logs/cored/confD.log'

# Database logs
c4 connect -i PLAY_ID -s cos -- 'cat /exa/logs/cored/db_MY_DATABASE.log'

# System logs
c4 connect -i PLAY_ID -s cos -- 'journalctl -u cored -n 100'
```

### Check Database State

```bash
c4 connect -i PLAY_ID -s cos -- 'confd_client db_state db_name: MY_DATABASE'
```

### List Backups

```bash
c4 connect -i PLAY_ID -s cos -- 'confd_client db_backup_list'
```

### Check Cluster Info

```bash
c4 connect -i PLAY_ID -s cos -- 'confd_client cluster_state'
```

### Test Database Connection

```bash
# From jump host
exaplus -c HOSTNAME:8563 -u sys -p exasol -sql "SELECT * FROM EXA_ALL_USERS"
```

---

## Advanced Debugging

### Enable Verbose Output

```bash
c4 -v ps -p PLAY_ID
c4 -vv apply -P -i ./config  # Even more verbose
```

### Check AWS Resources

```bash
# List instances by PLAY_ID
aws ec2 describe-instances --filters "Name=tag:play_id,Values=PLAY_ID"

# List volumes
aws ec2 describe-volumes --filters "Name=tag:play_id,Values=PLAY_ID"

# Check security groups
aws ec2 describe-security-groups --filters "Name=tag:play_id,Values=PLAY_ID"
```

### Network Troubleshooting

```bash
# From instance
c4 connect -i PLAY_ID -s cos -- 'ping -c 3 8.8.8.8'
c4 connect -i PLAY_ID -s cos -- 'traceroute exasol.com'
c4 connect -i PLAY_ID -s cos -- 'nslookup exasol.com'
```

---

## Getting Help

### Community Resources

- [Exasol Community](https://community.exasol.com/)
- [Exasol Documentation](https://docs.exasol.com/)
- [GitHub Issues](https://github.com/exasol/c4/issues)

### Support Channels

1. **Exasol Support Portal**
   - Log in with customer credentials
   - Submit support ticket
   - Include: PLAY_ID, error messages, logs

2. **Include in Support Request**:
   - c4 version: `c4 --version`
   - Error messages (full output)
   - Configuration file (sanitized)
   - Recent operations performed
   - AWS region and account ID
   - Deployment PLAY_ID

---

## Best Practices for Troubleshooting

**Collect information first**
- Error messages
- Recent operations
- Log files
- AWS console state

**Check basics**
- Credentials valid
- Network connectivity
- Service status
- Resource availability

**Try simple solutions first**
- Retry operation
- Increase timeout
- Restart service
- Check documentation

**Document issues**
- What was attempted
- Error messages
- Resolution steps
- Prevention measures

**Test in non-production first**
- Reproduce issue
- Verify solution
- Document process

---

## Related Documentation

- [c4 Overview](c4_overview.md)
- [c4 Configuration](c4_configuration.md)
- [c4 Timeout Configuration](c4_timeouts.md)
- [c4 MFA](c4_mfa.md)
- [c4 Best Practices](c4_best_practices.md)
- [Exasol Documentation](https://docs.exasol.com/)
