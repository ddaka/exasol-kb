---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Dead Node Monitor"
summary: "An alert was created as follows:"
---
# Dead Node Monitor

## Problem

An alert was created as follows:

Case Subject:  Dead Node Monitor

Case Description:

- n00xx.c0001.exacluster.local: state DEACTIVATED
- n00xx.c0001.exacluster.local: state FAILED

## Procedure

On Tags field, we can find Cluster ID, Account Group, Database Name and Node affected.

This alerts tells us that the affected is on Failed state or Deactivated.

Action:

1) Check status of the node
2) Check status of the database
3) If the status of the database is Down and there is a reserve node available, start the database and move the segment data to the new node
4) For Failed state: Check the reason why the node is on Failed state (i.e hardware problems, network problems etc)
5) For deactivated state: It usually happens when there is a maintenance on this particular node and this is deactivated on purpose.

## Additional References


