---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "IDRAC Global PSU state"
summary: "An alert was created as follows:"
---
# IDRAC Global PSU state

## Problem

An alert was created as follows:

Case Subject: IDRAC Global PSU state

Case Description:

- Please check PSU X

## Procedure

This alerts shows us that the one of the PSU on the nodes have errors. On Tags field, we can find Cluster ID, Account Group, Database Name and the node that is having PSU errors.

Action:

For the appliances clusters that we have access:

1) We need to check the status of the nodes.
2) We need to check the status of the PSU via check omsa tools&iDRAC
3) Generate the TSR Report to open the DELL case
4) Open the DELL case and schedule the PSU replacement

For appliances clusters that we do not have access:

1) Ask the customer to provide the TSR Report.
2) Open the DELL case and schedule the PSU replacement

For clusters that we only monitor:
1) inform the customer that a PSU failed and the PSU needs to be replaced.

## Additional References


