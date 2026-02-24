---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Node Uptime Monitor"
summary: "An alert was created as follows:"
---
# Node Uptime Monitor

## Problem

An alert was created as follows:

Case Subject: Node Uptime Monitor

Case Description:

- Host probably restarted a few minutes ago
## Procedure

This alert shows that the node is restarted a few minutes ago. On Tags field, we can find details for Cluster ID, Account Group, Database Name and the node that is restarted.

The action that needs to be taken:  we need to check the status and the health of the node that is restarted, the reason that caused the node to be restarted.

If there is hw issue and the node is an active data node, the node should be swapped with the reserve node and the issue should be fixed before the node is reintregrated to the  cluster.

## Additional References


