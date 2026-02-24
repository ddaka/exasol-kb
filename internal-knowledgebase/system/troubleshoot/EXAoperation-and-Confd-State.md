---
tool_name: confd_client
doc_type: troubleshoot
category: system
title: "EXAoperation and Confd State"
summary: "An alert was created as follows:"
---
# EXAoperation and Confd State

## Problem

An alert was created as follows:

Case Subject: EXAoperation and Confd State

Case Description:

- EXAoperation/Confd OFFLINE or very slow

## Procedure

This alerts shows us that the EXAoperation is down. On Tags field, we can find Cluster ID, Account Group, Database Name effected.

Action:

1) Check the status of EXAoperation
2) Check the output of `/var/log/logd/Appserver.log`
3) Restart the EXAoperation by running `coskillall appserverd`
4) If step 3 does not fix the issue, kill the appserverd partition and start it up again with `cosexec --single-instance --auto-restart --auto-add -- $COS_DIRECTORY/libexec/appserverd`
5) If the issue is still there, stop/start cos service on license server. It will not effect the database availability
6) If this is still not fixing the issue, restoring the exaoperation database might help - [Internal - Restore/Recreate EXAoperation DB](/Environment-Management/internal-restore-recreate-exaoperation-db.md)

## Additional References


