---
tool_name: c4
doc_type: troubleshoot
category: system
title: "SaaS - CPU Usage"
summary: "There are several reasons your instance might fail health checks due to over-utilization. The following are three of the most common reasons your health check might fail due to..."
---
# SaaS - CPU Usage

## Overview

There are several reasons your instance might fail health checks due to over-utilization. The following are three of the most common reasons your health check might fail due to over-utilization of resources:

* The CPU utilization of your instance reached close to 100%
* The root device is 100% full and the instance became stuck while booting.
* The processes running on the instance used all its memory, preventing the kernel from running.

 We don't alert cpu spike, we alert when CPU usage is constantly high (>99%) in last two hours.
## Procedure

### Step 1

Connect via AWS Console or CLI, follow the link: [this article](saas-login-to-sso.md)

### Step 2

* Sanity check on the environment:
	+ Is critical CPU usage occurred only one node or on all nodes?
	+ Which node does have a high critical cpu usage?
	+ Does this node has any other issue? Such as disk (check [disk status](saas-disk-status.md) or [disk utilization](saas-disk-usage-root-and-home-disks.md) issue, [swap issue](saas-high-swap-usage.md), [node health status issue](saas-host-health-check.md)
	+ If everything is fine from instance level continue on step 3

### Step 3

### **Tools for System Troubleshooting:**

There are a few tools that can help realize and troubleshoot CPU Load issues. Such as:

1. uptime
2. top
3. lsof

**Uptime:**

```sql
uptime: Uptime provides information about how long the system is up, the number of active users, and the load averages in the span of 1 minute, 5 minutes, and 15 minutes.
The last three numbers help you understand if the usage spike is long-term or short-term. The decimal represents the number of active tasks requesting CPU resources to perform an action. If the last number is too high, then that is a problem that should be taken care of.
```

![](images/exaEdis_0-1637853590144.png)

To understand and analyze numbers we should know how many processors are on the machine.

```sql
ubuntu@ip-192-168-146-39:~$ cat /proc/cpuinfo | grep processor
processor       : 0
processor       : 1
```

From the metrics above it can be understood that in the past 15 minutes each of the CPUs was used 99% of the time (1.98/2)

### Step 4

Ssh to container (for more info follow steps in this [link](saas-how-to-connect-to-the-container.md) and check processes using `top` command.

```sql
Running top provides a rich and self-updating layout of process information. top gives the load averages similar to the uptime output and it also includes per CPU metrics when you type ‘1’ after running top command.
```

![](images/exaAli_0-1643283364853.png)

Type `c` to see the related command for that process. Take process id and use `lsof` command to check which files access and used by this process. You can grep logs and find where this process writes log files:

```sql
# lsof -p <PID> | grep logs
```

 Check log files of this process. Do you see anything obvious on this log files?

If it is night and if high cpu usage creates general unavailability, you can restart the database and/or service that creates an issue. If it is business hours contact Production Engineering team.

Collect log files & provide to development:

* Collect logs and create a support ticket (for more info follow steps in this [link](how-to-get-log-files-from-exasol-saas-systems-temporary.md)
* If you are unable to ssh to the container in database/cluster instances due to c4 or c4_cloud_command issues, please follow this [link](collect-logs-from-no-container-running-clusters-on-saas.md) to collect logs when container is not running in customer database/cluster

## Additional References

[Database availability](saas-customer-database-cluster-availability-monitor.md)
