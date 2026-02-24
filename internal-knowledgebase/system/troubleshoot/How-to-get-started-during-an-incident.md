---
tool_name: cos
doc_type: troubleshoot
category: system
title: "How to get started during an incident"
summary: "This article describes the first steps you should do during an incident."
---
# How to get started during an incident

## Overview

This article describes the first steps you should do during an incident.

## Explanation

### Get an overview of the cluster health

Before starting with the troubleshooting do the following checks, even if you already know whats the issue might be.

- Check if EXAoperation is working normal (all pages are working, repsonse times OK, etc.)
- If the issue can be fixed from within EXAoperation do not use the shell otherwise
- SSH into the cluster
    - psh uptime (check the load and uptime for all data nodes)
    - psh ping -c1 license (check who is the EXAoperation master)
    - dwad_client list (Is the database running, what were the last 16 events)
    - cosps -r or cosps -N (check what cluster processes exist and if cos communication is working, who is online/offline, test from different nodes)
    - csinfo -v (check if storage is responding)
    - csrec -l (are recoveries running)
    - psh df -h (quickly check if there is no full disk)
    - Quickly check dmesg on suspects
    - check DB logs: logd_collect EXASolution_<DB_NAME>
    - check Storage logs: logd_collect Storage
    - check EXAoperation logs: logd_collect EXAoperation

If you can not fix the issue in 1 hour, escalate to the next person and get help in fixing the issue.


