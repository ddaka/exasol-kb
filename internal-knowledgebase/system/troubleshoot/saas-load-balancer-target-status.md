---
tool_name: c4
doc_type: troubleshoot
category: system
title: "SaaS - Load Balancer Target Status"
summary: "Load balancer node routes request only to the healthy targets in the enabled Availability Zones for the load balancer."
---
# SaaS - Load Balancer Target Status

## Overview

Load balancer node routes request only to the healthy targets in the enabled Availability Zones for the load balancer.

Each load balancer node checks the health of each target, using the health check settings for the target groups with which the target is registered. After your target is registered, it must pass one health check to be considered healthy. After each health check is completed, the load balancer node closes the connection that was established for the health check.

Load Balancer Target Status monitor check events of Instances every 15 minutes, if Port 4444 or instance is down it will alert us.

If load balancer does not work it means customers can not use API. Check cloud.exasol.com website and API to verify if they are reachable. If they are not reachable, communicate incident in relevant channels.

## Prerequisites

* Access to AWS Console, follow the link: [this article](saas-login-to-sso.md)

### Step 1

Check the status of the Host from Load Balancer

Open the Amazon EC2 console

* On the navigation pane, under **LOAD BALANCING**, choose **Target Groups**.
* Choose the name of the target group to open its details page.
* On the **Targets** tab, the **Status** column indicates the status of each target.
* If the status is any value other than `Healthy`, the **Status details** column contains more information

### Step 2

* Connect to the instance -> [link](saas-how-to-connect-to-the-instance.md)

### Step 3

* Check the Status of c4 -> [link](saas-c4-status-check.md)

### Step 4

* Check Database status

```sql
$ ubuntu@ip-192-168-2-18:~$ sudo lsof -nP -iTCP:8563 -sTCP:LISTEN
lsof: no pwd entry for UID 500
COMMAND     PID     USER   FD   TYPE     DEVICE SIZE/OFF NODE NAME
lsof: no pwd entry for UID 500
exacs   3200174      500   65u  IPv4 2528753678      0t0  TCP *:8563 (LISTEN)
```

* Check state for services **c4.service andc4_cloud_command** :
	+ `$ systemctl status c4_cloud_command`
	+ `$ systemctl status c4.service`
* If the status is other than `active` for c4.service and `activating` for c4_cloud_command follow the [link](saas-c4-status-check.md)

### Step 5

Find the PID of the WebUI-backend

```bash
$ ps -aux
root    1246631  0.1  0.2 750452 69172 ?  Ssl  Aug13  38:32 /usr/opt/EXASuite-7/EXAClusterOS-7.2.0-dev1/share/webui_backend/saas start --addr :4444 --rpc-endpoint https://localhost:20003 --signup-url https://<url>.e
```

* Kill the Process ID
* `kill -9 PID`

After the successful restart please double-check:

1. Web UI
2. Datadog Status
3. Dashboard
4. Curl the URL path

Follow step 6 only if steps 3 - 5 did not resolve the issue.

### Step 6

* Check Cpu Usage -> [link](saas-cpu-usage.md)
* Check High Swap Usage -> [link](saas-high-swap-usage.md)

Please contact Production Engineering Team if none of the steps above resolved the issue.

## Additional References

Here I link to other sites/information that may be relevant.

[Host Health Check](saas-host-health-check.md)

[Disk Status](saas-disk-status.md)
