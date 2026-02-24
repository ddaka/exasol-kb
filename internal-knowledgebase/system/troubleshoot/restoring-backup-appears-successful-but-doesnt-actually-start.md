---
tool_name: confd_client
doc_type: troubleshoot
category: system
title: "Restoring Backup Appears Successful but Doesn't Actually Start"
summary: "A remote archive was added to the deployment and a backup restore operation was initiated. Although the command appears to complete successfully, the database does not begin the..."
---
# Restoring Backup Appears Successful but Doesn't Actually Start

## Problem

A remote archive was added to the deployment and a backup restore operation was initiated. Although the command appears to complete successfully, the database does not begin the actual restore process.

There are no error messages presented. When checking the status of the restore, it immediately reports as **100% complete**, just seconds after initiating the command. This premature completion suggests the restore process did not actually start.

Customers may be affected if:

- A restore completes "too quickly"
- The restore status is shown as 100% with no activity or data restored

## Procedure

1. **Verify network access to the backup source**
   Ensure the remote archive is reachable from all cluster nodes using `ping` or `telnet`.
   Note: Telnet only works from the host and not from cos so test the connection from the host.

2. **Collect diagnostic logs**
   Ask the customer to reproduce the issue and provide:
   - ClusterOS logs
   - Server logs
   Make sure they add the `verbose` option in the remote volume to reproduce the issue.
   Please don't edit the EXAConf file and use the confd command to remove the backup and recreate it by referencing Exasol official documentation

3. **Validate remote volume configuration**
   Confirm that the remote volume is correctly configured in `/exa/etc/EXAConf` and that there are no syntax issues or missing options.

4. **Review the Pdd logs**
   Analyze the Pdd logs shared by the customer — these typically reveal the reason why the restore process silently fails.
   Pdd logs are usually found inside cos under: /exa/logs/db/`dbname`/

5. **Ensure consistent configuration between versions**
   When restoring a backup from a version 7 cluster to a version 8 cluster, confirm that the remote volume configuration is consistent across both environments (especially options like `nocompression`, etc.).
