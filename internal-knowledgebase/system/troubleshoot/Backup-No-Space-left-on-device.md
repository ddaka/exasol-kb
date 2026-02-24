---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Backup No Space left on device"
summary: "An alert was created as follows:"
---
# Backup No Space left on device

## Problem

An alert was created as follows:

Case Subject: Backup No Space left on device

Case Description:

- backups deleted in CLUSTER-XXX due to space restrictions
## Procedure

This alerts shows us that the backup on a local archive volume has failed due to no space left on the volume.

Action:

1) We need to check the status of the backup
2) Check if there is enough space to trigger a new backup
3) Check if there is any unused old backup that can be deleted in order to free up some space
4) Enlarge the archive volume if possible. Depending on the customer, we need permission to enlarge the archive volume
5) With the approval from customer, we can reduce the redundancy of the archive volume from 2 > 1 to gain free space
6) If we can not gain any space, we need to trigger a new one higher level backup(i.e a level-2 backup), and increase the expiration time of the parent backups. This is just a temporary solution, as the incremental backups can get very large.

## Additional References


