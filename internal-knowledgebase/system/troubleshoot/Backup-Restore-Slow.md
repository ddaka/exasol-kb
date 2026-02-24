---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Backup Restore Slow"
summary: "When restoring a backup to an AWS-based Exasol cluster that shares the same S3 remote volume with an on-premise cluster, customers may observe that the **restore process takes..."
---
# Backup Restore Slow

## Problem

When restoring a backup to an AWS-based Exasol cluster that shares the same S3 remote volume with an on-premise cluster, customers may observe that the **restore process takes significantly longer than expected**. This can affect recovery time objectives and delay service availability.

### How to Determine if You're Affected

- You are restoring from a remote S3 volume to an Exasol cluster hosted in AWS.
- The same S3 volume is also used by an on-premise Exasol cluster.
- You notice slow performance during the restore process without clear resource bottlenecks.

## Procedure

Follow the steps below to investigate the issue:

### Step 1

Collect relevant logs from the customer environment, including:

- `EXAClusterOS`
- `server/sql`
For more information about log collection, please refer to the following link:
[Exasol Support Documentation](https://docs.exasol.com/db/latest/administration/on-premise/support.htm)

These logs help identify configuration mismatches or performance anomalies during the restore.

### Step 2

Verify that the **EC2 instances in the AWS cluster are located in the same region as the S3 bucket**.
Cross-region access can introduce significant latency and slow down the restore process.

### Step 3

Check that the **remote volume configuration is consistent** between the on-premise and AWS clusters. Specifically:

- Review all parameters related to the S3 volume.
- Ensure that the `cloud_backend_v2` parameter is present in the AWS cluster’s remote volume configuration.

### Step 4

Since both clusters use the same S3 volume, ensure that **no backup process is running on the on-premise cluster** during the restore operation on the AWS cluster.
Simultaneous operations on the same volume can lead to contention and degraded performance.

### Step 5

If the cause of the issue is still unclear:

- Escalate to **R&D** for further investigation.
- Request that the customer **opens an AWS Support case** to check for underlying S3 or network-related performance issues on the AWS side.
