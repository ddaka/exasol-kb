---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "TCP Checksum Errors IN today vs. yesterday"
summary: "An alert was created as follows:"
---
# TCP Checksum Errors IN today vs. yesterday

## Problem

An alert was created as follows:

Case Subject:  TCP Checksum Errors IN today vs. yesterday

Case Description:

- At least 30 error rate increase
-

## Procedure

On Tags field, we can find Cluster ID, Account Group, Database Name affected.

This alerts indicates that there are some network problems on the affected node.

Action:

1) Check the status of the network and interfaces
2) Check the RX/TX errors with `ifconfig -a`

## Additional References


