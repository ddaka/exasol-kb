---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Telegraf monitor"
summary: "An alert was created as follows:"
---
# Telegraf monitor

## Problem

An alert was created as follows:

Case Subject:  Telegraf monitor

Case Description:

- Agent crit

## Procedure

On Tags field, we can find Cluster ID, Account Group, Database Name and Node affected.

This alerts tells us that the proust agent on the affected is has failed and the node is not being monitored.

Action:

1) Check status of the node
2) If the node is down, check the rca and reintegrate it into the cluster. Agent will be started automatically
3) There are cases when the connection fails due to temporary network issues. Wait for sometime if this is recovered by itself
4) Check if the routes are set up correctly for this node.
5) If the issue still persist, restart the proust partition.

## Additional References


