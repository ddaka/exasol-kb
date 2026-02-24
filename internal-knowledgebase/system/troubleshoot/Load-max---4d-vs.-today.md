---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Load max - 4d vs. today"
summary: "An alert was created as follows:"
---
# Load max - 4d vs. today

## Problem

An alert was created as follows:

Case Subject: Load max - 4d vs. today
Case Description: At least 100% higher Load

## Procedure

Check the hardware of the cluster.
Additionally, in `/var/log/sa/sarXX` you can find the CPU usage from last days.
Check the active session limit, involve database support to check for unusual queries.


