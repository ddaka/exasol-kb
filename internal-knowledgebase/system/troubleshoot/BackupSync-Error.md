---
tool_name: cos
doc_type: troubleshoot
category: system
title: "BackupSync Error"
summary: "An alert was created as follows:"
---
# BackupSync Error

## Problem

An alert was created as follows:

Case Subject:  BackupSync Error

Case Description:

- BackupSync Process Error on host n00XX CLUSTER-XXX

## Procedure

On Tags field, we can find Cluster ID, Account Group, Database Name and the node affected.

This alerts tells us that the synchronization of backups using backupsync plugin has failed.

Action:

1) Check the status of the synchronization, if there is any error:

    ```shell
    logd_collect EXAoperation | grep backup
    ```

2) Check if the backupsync&mbuffer processes are still running
3) It happens often that there might be zombie processes for backupsync&mbuffer on source and destination clusters. Please, kill all the processes and stop/start the backupsync again via:

    ```shell
    psh /usr/opt/EXAplugins/Administration.BackupSync-1.0.17/exaoperation-gate/status
    psh /usr/opt/EXAplugins/Administration.BackupSync-1.0.17/exaoperation-gate/deactivate
    psh /usr/opt/EXAplugins/Administration.BackupSync-1.0.17/exaoperation-gate/activate Name_from_sync.cfg
    ```

4) Check if the synchronization is started, for example using the iftop network tool

For more information, refer to the [Install Backup Synchronization Plugin](https://github.com/exasol/public-knowledgebase/blob/main/Environment-Management/install-backup-synchronization-plugin.md) article.

## Additional References


