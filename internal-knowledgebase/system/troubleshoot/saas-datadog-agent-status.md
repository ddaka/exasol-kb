---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "SaaS - Datadog Agent Status"
summary: "In this article, we explain how to troubleshoot the Datadog agent.## Prerequisites"
---
# SaaS - Datadog Agent Status

In this article, we explain how to troubleshoot the Datadog agent.## Prerequisites

Access to Datadog

### Step 1

Click on Monitors

![](images/exaEdis_0-1638268405448.png)

### Step 2

Click Check Summary

![](images/exaEdis_1-1638268553868.png)

### Step 3

Click on datadog.agent.check_status

![](images/exaEdis_3-1638268639288.png)

### Step 4

Find the Instance Id with Warn/Alarm Status from the list

![](images/exaEdis_4-1638268711243.png)

### Step 5

Connect via AWS Console or CLI, follow the link: [this article](saas-login-to-sso.md)

### Step 6

First, check datadog-agent status

`$ sudo datadog-agent status`

```sql
Please follow the commands below if the service status is not in <active (running)> state:
 $ sudo restart datadog-agent
```
### Step 7

If the Agent failed to start and no further information is provided, there are two ways to display logs for the Datadog Agent service.

### Option 1

 Use the following command to display all logs for the Datadog Agent service. If needed, use `-r` to print logs in reverse order.

`$ sudo journalctl -u datadog-agent.service`

### Option 2

1. Modify datadog.yaml file (path: /etc/datadog-agent/)
2. Replace log_level: INFO to log_level: DEBUG (plus uncomment)
3. Restart the Datadog Agent (sudo restart datadog-agent)
4. Check the logs -> /var/log/datadog

 If the issue still persists please contact the Production Engineer team.
