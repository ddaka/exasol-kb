---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "How to sync the time and timezone from the host to container"
summary: "While the time and the timezone are usually synced automatically when the Exasol namespace/container starts it sometimes is handy to know how this is done manually."
---
# How to sync the time and timezone from the host to container

## Overview

While the time and the timezone are usually synced automatically when the Exasol namespace/container starts it sometimes is handy to know how this is done manually.

Keep in mind there is no setting in EXAConf to set a NTP as this is managed through the host that runs the Exasol software.

## Explanation

```shell
scp -P20002 /etc/localtime root@localhost:/etc/localtime
### repeat this step for all nodes
### CAUTION cos_sync_files does not work! Copy each node one by one
```
