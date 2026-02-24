---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Link monitor"
summary: "An alert was created as follows: Case Subject: Link monitor"
---
# Link monitor

## Problem

An alert was created as follows:
Case Subject:   Link monitor

Case Description:

- Link down privateX
- Link down publicX

## Procedure

On Tags field, we can find Cluster ID, Account Group, Database Name and the node affected.

This alerts shows us that the interface on the node from Tag field is down.

Action:

1) Check status of the interfaces with command:

```ip a s```

2) Try to bring up the interface with ifup command or `ifconfig` command (`ifconfig interfacename up`).

* `up` argument brings the interface up.

3) If the status is still down, please check the output of: `dmesg`.

4) Check if the link is detected with:

```ethtool name_of_interface```

6) If Link detected:

- If it's ***Yes***, than everything is running as expected.
- It it's ***No***, it usually indicates that the cable is damaged or is not plugged in properly.
