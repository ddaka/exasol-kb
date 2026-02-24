---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Needed basis backup missing"
summary: "An alert was created as follows:"
---
# Needed basis backup missing

## Problem

An alert was created as follows:

Case Subject: Backup No Space left on device

Case Description:

- Needed basis backup missing CLUSTER-XXX
## Procedure

In general, this alerts shows us that the backup initializiation failed because the parent backup is missing.

The action:

1) We need to check and find out which backup is missing
2) If the a level_1 backup failed because of missing level_0 backup, we need to trigger a level_0 backup and have a usable level_0 backup available before triggering a new level_1 backup
3) If the a level_2 backup failed because of missing level_1 backup, we need to trigger a level_1 backup and have a usable level_1 backup available before triggering a new level_2 backup
4) The same goes for other backups

## Additional References


