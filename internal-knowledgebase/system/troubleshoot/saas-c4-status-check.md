---
tool_name: c4
doc_type: troubleshoot
category: system
title: "SaaS C4 Status Check"
summary: "In this article, we will show you how to check and fix c4 services."
---
# SaaS C4 Status Check

## Overview

In this article, we will show you how to check and fix c4 services.

## Prerequisites

AWS Credentials

### Step 1

Connect via AWS Console or CLI, follow the link: [this article](saas-login-to-sso.md)

### Step 2

Check c4.service status :

`$ systemctl status c4.service`

![](images/exaEdis_0-1638369100672.png)

If the status is other than active (running), please follow the next steps

### Step 3

Check the for logs

`$sudo journalctl -u c4.service`

### Step 4

Restart the service

`$ sudo systemctl stop c4.service`

`$ sudo systemctl start c4.service`

```sql
Note:    Check the logs during the starting process (Step 3)
If the issue still persists, please create a Support ticket and contact the Production Engineer Team
```
### Step 5

Check c4_cloud_command.service

`$ sudo systemctl status c4_cloud_command.service`

![](images/exaEdis_0-1638369871757.png)

If the status is other than activating (start), please follow the next steps

### Step 6

Check for logs:

`sudo journalctl -u c4_cloud_command.service`

```sql
Note:    Six steps (Stage0 - Stage6) must be completed for the successful startup of the c4_cloud_command.service.
Please follow the logs for ERROR or any permission issues during the startup process.
If the problem persists, please contact the Production Engineer Team.
```
### Step 7

Restart the service **(!!! This will restart customer databases as well, if you are not sure about it. Don't restart))**

`$ sudo systemctl stop c4_cloud_command.service`

`$ sudo systemctl start c4_cloud_command.service --no-block`

```sql
Note:    Check the logs during the starting process (Step 6)
If the issue still persists, please create a Support ticket and contact the Production Engineer Team
```
