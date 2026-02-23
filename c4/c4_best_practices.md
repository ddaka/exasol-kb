---
tool_name: c4
doc_type: guide
category: c4 Best Practices
title: "c4 Best Practices"
summary: "c4 apply -P -i ./config"
---
# c4 Best Practices

## Operational Best Practices

### Planning Deployments

**Size appropriately**
- Match instance types to workload
- Consider future growth
- Balance cost vs performance

**Choose regions wisely**
- Proximity to users
- Data sovereignty requirements
- AWS service availability

**Network planning**
- Dedicated VPC for databases
- Private subnets for data nodes
- Public subnet only for access node

**Documentation**
- Document deployment parameters
- Keep config files in version control
- Maintain runbooks

### Deployment Creation

**Use config files**
```bash
# Always use -i for repeatability
c4 apply -P -i ./config
```

**Version control configs**
```bash
git add config
git commit -m "Add production Exasol config"
git push
```

**Tag deployments**
- Use descriptive PLAY_IDs
- Add AWS tags for organization
- Document purpose

**Test first**
- Create test deployment
- Validate configuration
- Verify functionality
- Then create production

### Naming Conventions

**Deployment names**
```
Format: <environment>-<purpose>-<region>
Examples:
  prod-analytics-us-east-1
  dev-testing-eu-west-1
  stage-etl-ap-south-1
```

**Database names**
```
Format: <application>_<environment>
Examples:
  analytics_prod
  reporting_dev
  datawarehouse_stage
```

**Config files**
```
Format: config-<environment>-<cluster-size>n
Examples:
  config-prod-5n
  config-dev-3n
  config-stage-10n
```

---

## Security Best Practices

### Access Control

**Least privilege principle**
- Grant minimum required permissions
- Use IAM roles instead of keys when possible
- Regular access reviews

**MFA enforcement**
```bash
# Always use MFA for production
aws sts get-session-token \
  --serial-number arn:aws:iam::123456789012:mfa/username \
  --token-code 123456
```

**SSH key management**
- One key per environment
- Rotate keys regularly
- Store securely (never in git)
- Use SSH agent forwarding

**Network security**
- Restrict security group rules
- Use VPN for database access
- No public IPs for data nodes
- Audit security group changes

### Credential Management

**Never commit credentials**
```bash
# Add to .gitignore
echo "*.pem" >> .gitignore
echo "credentials" >> .gitignore
echo ".aws" >> .gitignore
```

**Use AWS Secrets Manager**
- Store database passwords
- Rotate credentials regularly
- Audit secret access

**Environment variables**
```bash
# Set in shell, not in scripts
export AWS_ACCESS_KEY_ID="..."
export AWS_SECRET_ACCESS_KEY="..."
```

**Credential rotation**
- Rotate AWS keys quarterly
- Rotate database passwords monthly
- Update documentation

---

## Maintenance Best Practices

### Backups

**Regular backups**
```bash
# Daily automated backups
c4 connect -i PLAY_ID -s cos -- 'confd_client db_backup_add_schedule \
  db_name: MY_DATABASE \
  enabled: true \
  hour: 2 \
  minute: 0 \
  level: 0'
```

**Backup retention**
- Production: 30 days minimum
- Development: 7 days
- Testing: 3 days

**Test restores regularly**
```bash
# Monthly restore test
c4 connect -i PLAY_ID -s cos -- 'confd_client db_backup_list'
c4 connect -i PLAY_ID -s cos -- 'confd_client db_restore backup_id: BACKUP_ID'
```

**Off-site backups**
- Copy to S3 in different region
- Test cross-region restore
- Document restore procedures

### Updates

**Regular patching**
- Apply security patches promptly
- Test updates in dev/staging first
- Schedule maintenance windows

**Update procedure**
```bash
# 1. Backup
c4 connect -i PLAY_ID -s cos -- 'confd_client db_backup_start db_name: MY_DATABASE'

# 2. Stop database
c4 connect -i PLAY_ID -s cos -- 'confd_client db_stop db_name: MY_DATABASE'

# 3. Update
CCC_PLAY_TIMEOUT=1h c4 update cluster -p PLAY_ID -t @exasol-2025.1.0

# 4. Start database
c4 connect -i PLAY_ID -s cos -- 'confd_client db_start db_name: MY_DATABASE'

# 5. Verify
c4 connect -i PLAY_ID -s cos -- 'confd_client db_state db_name: MY_DATABASE'
```

**Version tracking**
- Document current versions
- Track update history
- Note any issues encountered

**Rollback plan**
- Keep previous version available
- Document rollback procedure
- Test rollback in staging

### Monitoring

**Regular health checks**
```bash
# Daily status check
c4 ps -p PLAY_ID

# Database state
c4 connect -i PLAY_ID -s cos -- 'confd_client db_state db_name: MY_DATABASE'

# Disk usage
c4 connect -i PLAY_ID -s cos -- 'df -h'
```

**Log management**
- Centralize logs (CloudWatch, ELK)
- Set retention policies
- Monitor for errors
- Alert on critical events

**Performance monitoring**
- Track query performance
- Monitor resource utilization
- Set up alerts for anomalies
- Regular capacity reviews

**Automated monitoring**
- Use CloudWatch alarms
- Configure SNS notifications
- Monitor AWS service health
- Track cost and usage

---

## Cost Optimization

### Right-Sizing

**Instance sizing**
- Monitor actual usage
- Downsize overprovisioned instances
- Use Reserved Instances for production
- Spot Instances for dev/test

**Storage optimization**
- Regular cleanup of old data
- Compress infrequently accessed data
- Use lifecycle policies for backups
- Monitor EBS usage

**Network costs**
- Keep traffic within same region
- Use VPC endpoints
- Monitor data transfer costs

### Resource Cleanup

**Remove unused deployments**
```bash
# List all deployments
c4 ls

# Remove old ones
c4 rm -p OLD_PLAY_ID
```

**Stop dev/test when not in use**
```bash
# Stop database
c4 connect -i PLAY_ID -s cos -- 'confd_client db_stop db_name: MY_DATABASE'

# Stop nodes
c4 down PLAY_ID
```

**Clean up orphaned resources**
- Unattached EBS volumes
- Unused Elastic IPs
- Old snapshots
- Unused security groups

**Cost tracking**
- Tag all resources
- Enable cost allocation tags
- Regular cost reviews
- Set up budget alerts

---

## Disaster Recovery

### DR Planning

**RTO/RPO definition**
- Recovery Time Objective
- Recovery Point Objective
- Document for each system
- Test regularly

**Multi-region strategy**
- Primary and DR regions
- Automated backups to DR region
- Regular DR drills
- Document procedures

**Backup verification**
```bash
# Monthly DR test
# 1. Create deployment in DR region
c4 apply -P -i ./config-dr

# 2. Restore backup
c4 connect -i DR_PLAY_ID -s cos -- 'confd_client db_restore backup_id: BACKUP_ID'

# 3. Verify data
# 4. Document results
# 5. Clean up
c4 rm -p DR_PLAY_ID
```

### Backup Strategy

**3-2-1 rule**
- **3** copies of data
- **2** different storage types
- **1** off-site copy

**Backup types**
- Full backups: weekly
- Incremental: daily
- Transaction logs: real-time

**Testing schedule**
- Weekly: Backup completion verification
- Monthly: Restore test in dev
- Quarterly: Full DR drill

---

## Documentation Best Practices

### Essential Documentation

**Deployment inventory**
```markdown
# Production Deployments

| PLAY_ID | Environment | Region | Size | Database | Purpose |
|---------|-------------|--------|------|----------|---------|
| abc123  | prod        | us-east-1 | 5 nodes | analytics_prod | Analytics |
| def456  | stage       | us-west-2 | 3 nodes | analytics_stage | Staging |
```

**Runbooks**
- Common operations
- Troubleshooting steps
- Emergency procedures
- Contact information

**Configuration management**
- Version control all configs
- Document parameters
- Track changes
- Review process

**Incident log**
- Document all issues
- Resolution steps
- Lessons learned
- Prevention measures

### Knowledge Sharing

**Team training**
- Onboarding documentation
- Regular training sessions
- Knowledge transfer
- Cross-training

**Change management**
- Document all changes
- Peer review changes
- Communicate to team
- Update documentation

**Lessons learned**
- Document incidents
- Share solutions
- Update procedures
- Regular reviews

---

## Automation Best Practices

### Infrastructure as Code

**Config files in git**
```bash
# Maintain configs
git add config-prod-5n
git commit -m "Update node size to m5.2xlarge"
git push
```

**Automated deployments**
```bash
#!/bin/bash
# deploy.sh

ENVIRONMENT=$1
CONFIG="config-${ENVIRONMENT}"

# Deploy
c4 apply -P -i "./${CONFIG}"

# Verify
c4 ps -P
```

**CI/CD integration**
- Automated testing
- Deployment pipelines
- Validation steps
- Rollback capability

### Scripts and Tools

**Wrapper scripts**
```bash
#!/bin/bash
# c4-with-mfa.sh

read -p "MFA code: " MFA_CODE

# Get session token
aws sts get-session-token \
  --serial-number "${MFA_SERIAL}" \
  --token-code "${MFA_CODE}" \
  > /tmp/mfa.json

# Set credentials
export AWS_ACCESS_KEY_ID=$(jq -r '.Credentials.AccessKeyId' /tmp/mfa.json)
export AWS_SECRET_ACCESS_KEY=$(jq -r '.Credentials.SecretAccessKey' /tmp/mfa.json)
export AWS_SESSION_TOKEN=$(jq -r '.Credentials.SessionToken' /tmp/mfa.json)

# Run c4
c4 "$@"
```

**Health check scripts**
```bash
#!/bin/bash
# health-check.sh

PLAY_ID=$1

echo "Checking deployment ${PLAY_ID}..."
c4 ps -p "${PLAY_ID}"

echo "Checking database state..."
c4 connect -i "${PLAY_ID}" -s cos -- 'confd_client db_state db_name: MY_DATABASE'

echo "Checking disk usage..."
c4 connect -i "${PLAY_ID}" -s cos -- 'df -h /exa'
```

---

## Compliance and Governance

### Audit Trail

**Log all operations**
- Use CloudTrail
- Enable S3 access logs
- Database audit logs
- Regular reviews

**Change tracking**
- Document all changes
- Approval process
- Version control
- Audit reviews

**Access reviews**
- Quarterly IAM reviews
- Remove unused accounts
- Verify permissions
- Document exceptions

### Data Governance

**Data classification**
- Identify sensitive data
- Implement encryption
- Access controls
- Retention policies

**Compliance requirements**
- GDPR, HIPAA, SOC2
- Document compliance measures
- Regular audits
- Remediation tracking

---

## Emergency Procedures

### Critical Issues

**Database corruption**
1. Stop accepting new connections
2. Assess extent of damage
3. Restore from last good backup
4. Verify data integrity
5. Resume operations

**Security breach**
1. Isolate affected systems
2. Revoke compromised credentials
3. Assess impact
4. Restore from known good state
5. Implement additional controls

**Data loss**
1. Stop all write operations
2. Assess recovery options
3. Restore from backup
4. Verify data completeness
5. Document incident

### Contact Information

**Escalation paths**
- L1: Team lead
- L2: Engineering manager
- L3: Director / VP

**External contacts**
- Exasol Support
- AWS Support
- Security team
- Legal/Compliance

---

## Summary Checklist

### Daily Operations
- [ ] Check deployment status (`c4 ps`)
- [ ] Verify backup completion
- [ ] Review logs for errors
- [ ] Monitor resource usage

### Weekly Operations
- [ ] Backup verification
- [ ] Security group review
- [ ] Cost review
- [ ] Update documentation

### Monthly Operations
- [ ] Restore test from backup
- [ ] Patch/update review
- [ ] Access review
- [ ] DR drill (if scheduled)

### Quarterly Operations
- [ ] Full DR test
- [ ] Security audit
- [ ] Capacity planning review
- [ ] Documentation review

---

## Related Documentation

- [c4 Overview](c4_overview.md)
- [c4 Configuration](c4_configuration.md)
- [c4 Managing Nodes](c4_managing_nodes.md)
- [c4 Troubleshooting](c4_troubleshooting.md)
- [c4 MFA](c4_mfa.md)
- [c4 Timeout Configuration](c4_timeouts.md)
- [Exasol Security Best Practices](https://docs.exasol.com/db/latest/administration/on-premise/security_best_practices.htm)
