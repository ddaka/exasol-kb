---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Inconsistent Backup Expire Times"
summary: "An alert was created as follows:"
---
# Inconsistent Backup Expire Times

## Problem

An alert was created as follows:

Case Subject: Inconsistent Backup Expire Times

Case Description: Inconsistent Backup Expire Times

## Procedure

In general, this alert shows us that the expiration time of a running backup expires after the expiration time of the parent backup.

This means that the parent backup will be removed before the running backup, which causes the situation when the backup will not be usable anymore.

Action:

We need to adjust the expiration time accordingly, so the running backup expires before the parent backup.
## Additional References


