---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "SaaS Management instance Disk Status &amp; Usage &amp; Burst Balance"
summary: "Check EBS volume health, filesystem usage, and (legacy) gp2 burst balance for SaaS management instances."
---
# SaaS Management instance Disk Status &amp; Usage &amp; Burst Balance

## Scope note

Burst balance applies to `gp2` only. If volumes were migrated to `gp3`, burst balance checks are not relevant.

## Purpose

Validate disk health and capacity signals on SaaS management instances.

## Prerequisites

- Access to AWS Console and EC2/CloudWatch views.
- Access to connect to the management instance shell.

## Procedure

1. In EC2 console, open the management instance and inspect attached volumes from the `Storage` tab.

2. Open each volume and check:
- `Volume status`
- `State`
- Type (`gp2` or `gp3`)

3. On the instance, verify block devices:

```bash
lsblk
```

4. Check filesystem usage:

```bash
df -hT
```

5. If volume type is `gp2`, check burst balance in CloudWatch:
- `CloudWatch -> Metrics -> EBS -> Per-Volume Metrics -> BurstBalance`
- Consider alerting when balance remains low for extended periods.

## Related runbooks

- Root/home usage troubleshooting:
  [`../troubleshoot/saas-disk-usage-root-and-home-disks.md`](../troubleshoot/saas-disk-usage-root-and-home-disks.md)
- Disk status troubleshooting:
  [`../troubleshoot/saas-disk-status.md`](../troubleshoot/saas-disk-status.md)

## References

[View information about an Amazon EBS volume](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-describing-volumes.html)
[Monitor the status of your volumes](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/monitoring-volume-status.html)
[Burst Balance Metric for EC2’s General Purpose SSD (gp2) Volumes](https://aws.amazon.com/blogs/aws/new-burst-balance-metric-for-ec2s-general-purpose-ssd-gp2-volumes/)
