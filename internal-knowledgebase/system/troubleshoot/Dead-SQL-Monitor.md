---
tool_name: cos
doc_type: troubleshoot
category: system
title: "Dead SQL Monitor"
summary: "An alert was created as follows:"
---
# Dead SQL Monitor

## Problem

An alert was created as follows:

Case Subject:  Dead SQL Monitor

Case Description:

- SQL monitor for XXX CRIT

## Procedure

On Tags field, we can find Cluster ID, Account Group, Database Name affected.

This alerts tells us that our monitoring is having issues to access the database.

Action:

1) Check if Proust is updated with the latest data
2) Check the status of the database
3) Check if you are able to login to the database and run simple query
4) Check if the system have reached maximum active sessions
5) If the issues still persists, clean up old `check_session`
- Find the process with `cosps -N | grep session` or `cosps -N | grep stats`
- remove the processes from COS with `cosrm -a <PARTITIONID>`

6) If the issues still persists, inform the team via proust-feedback channel and silent the alert for some period of time

## Additional References


