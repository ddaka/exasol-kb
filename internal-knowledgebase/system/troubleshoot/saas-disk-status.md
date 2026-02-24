---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "SaaS - Disk Status"
summary: "Volume status checks enable you to better understand, track, and manage potential inconsistencies in the data on an Amazon EBS volume. They are designed to provide you with the..."
---
# SaaS - Disk Status

## Overview

Volume status checks enable you to better understand, track, and manage potential inconsistencies in the data on an Amazon EBS volume. They are designed to provide you with the information that you need to determine whether your Amazon EBS volumes are impaired, and to help you control how a potentially inconsistent volume is handled.

Volume status checks are automated tests that run every 5 minutes and return a pass or fail status. If all checks pass, the status of the volume is `ok`. If a check fails, the status of the volume is `impaired`. If the status is `insufficient-data`, the checks may still be in progress on the volume

## Prerequisites

* AWS Credentials
* Datadog Access

### Step 1

Get The volume id from Datadog Monitor

### Step 2

Connect via AWS Console or CLI, follow the link:[link](saas-login-to-sso.md)

### Step 3

1. Open the Amazon EC2 console.
2. In the navigation pane, choose **Volumes**.
3. Filter Volumes by Id.
4. Select a volume to view its specific event.

### Step 4

**Note:** You can't run fsck or do anything else safely unless you have backups of all the data you want to keep. It is possible to loose data after a run of fsck. Depending on if and how the file system is damaged files and/or folders can be lost. Therefore, take a snapshot before doing this check.

Follow the steps to take snapshot:

```sql
1. Open the Amazon EC2 console
2. In the navigation pane, choose Snapshots, Create snapshot.
3. For Resource type, choose Volume.
4. For Volume ID, select the volume from which to create the snapshot.
5. The Encryption field indicates the selected volume's encryption status. If the selected volume is encrypted, the snapshot is automatically encrypted using the same KMS key. If the selected volume is unencrypted, the snapshot is not encrypted.
6. (Optional) For Description, enter a brief description for the snapshot.
7. (Optional) To assign custom tags to the snapshot, in the Tags section, choose Add tag, and then enter the key-value pair. You can add up to 50 tags.
8. Choose Create snapshot.
```

### Step 5

1. Connect to the Instance: [link](saas-how-to-connect-to-the-instance.md)
2. Run the **fsck** command `fsck -y <volume>`
3. If the volume has been impaired for more than 20 minutes, you can contact the Production Engineering Team.

## Additional References

Login to SSO: [link](saas-login-to-sso.md)

Disk Usage: [link](saas-disk-usage-root-and-home-disks.md)
