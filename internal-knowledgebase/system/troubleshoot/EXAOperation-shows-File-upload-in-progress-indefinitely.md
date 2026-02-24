---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "EXAOperation shows File upload in progress... indefinitely under Configuration, Software, Versions"
summary: "If you upload software update file to v7 EXAOperation GUI, but for some reason the upload is stalled, then the EXAOperation shows **File upload in progress...** indefinitely under..."
---
# EXAOperation shows File upload in progress... indefinitely under Configuration, Software, Versions

## Problem

If you upload software update file to v7 EXAOperation GUI, but for some reason the upload is stalled, then the EXAOperation shows **File upload in progress...** indefinitely under Configuration, Software, Versions. This symptom persists even after you completely reboot all the nodes.

## Procedure

The cause of the symptom is presence of a flag file on the license node, which path is `/usr/opt/EXASuite-7/EXAClusterOS-<VERSION>/etc/file_upload_progress`. The EXAOperation GUI will come back to normal state after manually removing the flag file from the backend using root credentials.

How to enable root login on:
1. Login to SSH as maintenance user on the license node.
2. Go to Passwords.
3. Select Unlock root password, then hit Enter key.
4. Select Back.
5. Select Exit.

How to delete the flag file:
1. Login to SSH as root user on the license node.
2. Locate the flag file:
`ls /usr/opt/EXASuite-7/EXAClusterOS-<VERSION>/etc/file_upload_progress`
3. Ensure to terminate the stalled process, for example by rebooting all the nodes including license server, or executing below command if Oracle Instant Client upload is stalled:
`pkill -f -e oracle_upload`
4. Remove the flag file:
`rm /usr/opt/EXASuite-7/EXAClusterOS-<VERSION>/etc/file_upload_progress`
