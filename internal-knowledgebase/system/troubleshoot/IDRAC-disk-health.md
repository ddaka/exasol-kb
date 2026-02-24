---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "IDRAC disk health"
summary: "An alert was created as follows:"
---
# IDRAC disk health

## Problem

An alert was created as follows:

Case Subject: IDRAC disk health

Case Description:

- Physical Disk 0:1:7 state: FAILED

## Procedure

This alerts shows us that the one of the disk on the nodes have failed. On Tags field, we can find Cluster ID, Account Group, Database Name and the node that is having failed disk.

Action:

For the appliances clusters that we have access:

1) We need to check the status of the nodes.
2) We need to check the status of the disk via check omsa tools&iDRAC
3) Generate the TSR Report to open the DELL case
4) Open the DELL case and schedule the disk replacement

For appliances clusters that we do not have access:

1) Ask the customer to provide the TSR Report.
2) Open the DELL case and schedule the disk replacement

For clusters that we only monitor:
1) inform the customer that a disk failed and the disk needs to be replaced.

## Additional References


