---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Storage CRC Error"
summary: "An alert was created as follows:"
---
# Storage CRC Error

## Problem

An alert was created as follows:

Case Subject:  Storage CRC Error

Case Description: CRC Errors found

## Procedure

On Tags field, we can find Cluster ID, Account Group, Database Name and the node that is having CRC Errors.

Usually, this alert indicates that there is something wrong with the storage on the node that is showing the CRC Error.
See also: [Storage Disk Monitor](https://github.com/exasol/internal-knowledgebase/blob/main/Monitoring-Alerts/Storage-Disk-Monitor.md)

Action:

1) Check if there is any storage hardware issue on the node
2) Check if there was any storage hardware issue in the past (which might have caused the CRC errors)
3) If the CRC Errors are coming from a hw issue from the past, you can clear them from EXAOperation

Clear the errors only if you are sure that there are no more disk issues for it. For Dell servers check the disks with MegaCLI.

EXAStorge => n00X => Select the disk => Clear errors

## Additional References

[Storage Disk Monitor](https://github.com/exasol/internal-knowledgebase/blob/main/Monitoring-Alerts/Storage-Disk-Monitor.md)


