---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Swap in Use"
summary: "An alert was created as follows:"
---
# Swap in Use

## Problem

An alert was created as follows:

Case Subject: Swap in Use

Case Description:

Swap in use but NOT ACTIVE

## Procedure

On Tags field, we can find Cluster ID, Account Group, Database Name and the node that is having high usage of Swap Memory.
Swap space is the portion of virtual memory that is on the hard disk, used when RAM is full.
In general, in Exasol systems it happens when the Hugepages and OS Memory settings are not set correctly in EXAoperation.
You can find information about how to calculate the hugepages in our Knowledgebase article:

[INTERNAL - How to calculate hugepages](/Environment-Management/internal-how-to-calculate-hugepages.md)

In addition, you can still clear the swap if (swapoff & swapon):

```shell
/proc/meminfo that MemFree > SwapTotal - SwapFree
```

## Additional References


