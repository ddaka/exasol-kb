---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Node Monitor"
summary: "An alert was created as follows:"
---
# Node Monitor

## Problem

An alert was created as follows:

Case Subject: Node Monitor

Case Description:

- Node X failed
## Procedure

This alert shows that one of the nodes have failed. If this is a active data node, the database will crash and will be down. If there is a reserve node available, the reserve node will take over the role from the faulty node and the failover will be triggered.
We should check status of the node, the reason of the failure (if this is a hardware issue, network issue or the node is deactivated) and bring it up as soon as possible.

## Additional References


