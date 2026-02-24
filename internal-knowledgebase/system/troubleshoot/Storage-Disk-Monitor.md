---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Storage Disk Monitor"
summary: "An alert was created as follows:"
---
# Storage Disk Monitor

## Problem

An alert was created as follows:

Case Subject:  Storage Disk Monitor

Case Description: /dev/disk_d03_storage_X state: OFFLINE

## Procedure

On Tags field, we can find Cluster ID, Account Group, Database Name and the node that is having one of the disks in OFFLINE state.

Usually, this alert indicates that there are some hw errors with the disk or controller. An immediate reaction is needed, as it can cause database crash.
Volumes with Redundancy 1 will be locked and can not be used until the disk online again. Volumes with a higher Redundancy will be in the status 'degraded' and can still be used.
Locked volumes can not be used by the database, this means as soon as the database tries to write into one, the database will restart or crash.

In this case it is highly recommended to swap the affected node immediately with the reserve node.

If this is not possible at the moment (customer not reachable in a timely manner), check the disk health of the affected node (Dell: MegaCLI).
In case of a RAID and at least one functional disk is available in this RAID (no error count), set the faulty disk offline.
Afterwards, enable the disk in EXAoperation again:
EXAoperation -> EXAStorage -> n00X -> enable devices

Since the RAID is now running with one less disk offer the customer the following options:
- swap node with reserve node (recommended since the disk was taken offline in EXAoperation)
- replace the disk as usual

## Additional References

[Storage CRC Error](https://github.com/exasol/internal-knowledgebase/blob/main/Monitoring-Alerts/Storage-CRC-Error.md)


