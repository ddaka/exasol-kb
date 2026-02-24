---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Storage IO Error Monitor today vs yesterday"
summary: "An alert was created as follows:"
---
# Storage IO Error Monitor today vs yesterday

## Problem

An alert was created as follows:

Case Subject: Storage IO Error Monitor today vs yesterday
Case Description: Check: Storage IO Error Monitor today vs yesterday: today: 1 yesterday: 0

## Procedure

Check the hardware of the cluster.
For Dell systems, with the `check_openmanage` in `/opt/` and with MegaCLI for example: `MegaCli64 -pdList -aall | grep -E  "Error|Device|state|Predictive"`.
In EXAoperation check for offline disks and for IO errors on the nodes itself (EXAStorage -> nodeXXX).


