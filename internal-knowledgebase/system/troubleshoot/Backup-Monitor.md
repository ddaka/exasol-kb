---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Backup Monitor"
summary: "An alert was created as follows:"
---
# Backup Monitor

## Problem

An alert was created as follows:

Case Subject: Backup Monitor

Case Description:

- Backup: ABORTED
- Backups failed within the last 24h

## Procedure

Backup aborted:

The alerts with this description tells that the backup has failed for some reason.
The action to check:

1) Check the status of the backup - You can check this in EXASolution tab of EXAoperation
2) Check if the backup was running on a local or remote volume
3) Try to find out the reason of the backup failure:

- If this is a local backup, check the state of the volume.
- If this is a remote volume, check the status of the remote volume, if the connection to the remote volume is working. If possible, check if there is enough storage space on the remote volume.

4) Set the verbose option and trigger a new backup again if the state of the volume or status of the remote volume is Online.
5) If the backup succeeds, then remove the verbose option again, as it will generate unnecessary logs and the log file might be big in size.
If the backup fails again, please get the logs and transfer them to R&D for investigation.

Backups failed within the last 24h:

This alert tells us that a backup failure happened in the last 24 hours.
We need to check the status of the last backups, and check if the new backup is triggered.

Also, we should double check that the customer is informed properly for the failed backups.

## Additional References


