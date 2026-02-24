---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Storage Monitor"
summary: "An alert was created as follows:"
---
# Storage Monitor

## Problem

An alert was created as follows:

Case Subject: Storage Monitor

Case Description:

- Storage OFFLINE
## Procedure

On Tags field, we can find Cluster ID, Account Group, Database Name and Node affected.

This alerts indicates that the Storage on the cluster is OFFLINE. This can be caused by HW issues, network issues etc,

Action:

1) Check status of the nodes, storage cluster and database
2) Check if there are any storage hw issues on the nodes
3) Check the network communication between the node
4) If the storage is OFFLINE, start it up and start the database

## Additional References


