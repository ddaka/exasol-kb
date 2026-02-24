---
tool_name: internal-knowledgebase
doc_type: reference
category: system
title: "What is a virtual restore?"
summary: "This article explains what a virtual restore is, discusses its use cases, and outlines its advantages and disadvantages."
---
# What is a virtual restore?

## Overview

This article explains what a virtual restore is, discusses its use cases, and outlines its advantages and disadvantages.

## Explanation

A virtual restore is a method of reading and recovering the contents of a backup without the need of a full restore.

This is accomplished by adding an additional DB instance to your cluster and having the virtual restore read the backup.

A virtual restore is primarily used when tables or data gets removed from the backup.

Pros:
Virtual Restores are not limited to license limitations.
They use less resources then a full restore and can be removed when the restore is complete.

Cons:
Virtual Restore's do not work with remote archive volumes.
You need to allocate some resources (RAM, disk space, CPU) from your cluster.

## Additional References

https://docs.exasol.com/db/7.1/administration/on-premise/backup_restore/virtual_access_on_backup.htm


