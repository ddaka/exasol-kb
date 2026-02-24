---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Playbook: Backupstart - volume cannot be chosen"
summary: "This article shows how to solve the situation if a volume is not choosable when starting a backup manually or creating a backup scheduler for Exasol databases via EXAoperation."
---
# Playbook: Backupstart - volume cannot be chosen

## Overview

This article shows how to solve the situation if a volume is not choosable when starting a backup manually or creating a backup scheduler for Exasol databases via EXAoperation.

## Diagnosis

When starting a backup or creating a scheduler, the volume is not choosable.

 Starting a backup manually:

 ![](images/Manual_Backup.PNG)

 Creating a scheduler:

 ![](images/Scheduling.PNG)

 ## Explanation

Unfortunately, the user starting the backup or creating the scheduler does not have the rights to execute these tasks. This right needs to be set within the volume.

![](images/Archive_Volume1.PNG)

## Recommendation

Add this user to the volume as "Allowed User" or change the user.

![](images/Archive_Volume2.PNG)

## Additional References

Additional User rights are described within our docs. Please find further details at <https://docs.exasol.com/administration/on-premise/user_management/create_edit_and_delete_users.htm>
