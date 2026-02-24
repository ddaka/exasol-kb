---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Network Error Monitor today vs yesterday"
summary: "An alert was created as follows:"
---
# Network Error Monitor today vs yesterday

## Problem

An alert was created as follows:

Case Subject: Network Error Monitor today vs yesterday
Case Description: At least 15% increase

## Procedure

Check the network interfaces for example with `ifconfig` for errors,  `dmesg` for flapping/offline interfaces and with `ethtool` for offline links.
Probably there is a faulty cable or a faulty network interface on the server or switch.


