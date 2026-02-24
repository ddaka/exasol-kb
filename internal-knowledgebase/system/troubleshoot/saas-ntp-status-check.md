---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "SaaS - NTP Status Check"
summary: "The purpose of this article is to show how to troubleshoot NTP sync issue"
---
# SaaS - NTP Status Check

The purpose of this article is to show how to troubleshoot NTP sync issue
## Prerequisites

Access to Datadog

### Step 1

Click on Monitors

![](images/exaEdis_0-1637159807516.png)

### Step 2

Click Check Summary

![](images/exaEdis_1-1637159843521.png)

### Step 3

Click on ntp.in. sync

![](images/exaEdis_2-1637159882815.png)

### Step 4

Find the Instance Id with Warn/Alarm Status from the list

![](images/exaEdis_3-1637159953692.png)

### Step 5

Connect via AWS Console or CLI, follow the link: [this article](saas-login-to-sso.md)

### Step 6

First, check datadog-agent status

`$ sudo datadog-agent status`

```bash
    ntp
    ---
      Instance ID: ntp:<id> [OK]
      Configuration Source: file:/etc/datadog-agent/conf.d/ntp.d/conf.yaml.default
      Total Runs: 4
      Metric Samples: Last Run: 1, Total: 4
      Events: Last Run: 0, Total: 0
      Service Checks: Last Run: 1, Total: 4
      Average Execution Time : 1ms
      Last Execution Date : 2021-08-24 14:03:18 UTC (1629813798000)
      Last Successful Execution Date : 2021-08-24 14:03:18 UTC (1629813798000)
```

Check time sync with the command : `timedatectl status`

```bash
ubuntu@ip-192-168-32-36:~$ timedatectl status
              		 Local time: Mon 2021-09-06 10:26:46 UTC
           			Universal time: Mon 2021-09-06 10:26:46 UTC
                 		RTC time: Mon 2021-09-06 10:26:47
                		Time zone: Etc/UTC (UTC, +0000)
System clock synchronized: no
              	NTP service: active
          	RTC in local TZ: no
```

If the `System clock synchronized: no`

Check the Service Status

`$ systemctl status systemd-timesyncd.service`

```bash
● systemd-timesyncd.service - Network Time Synchronization
Loaded: loaded (/lib/systemd/system/systemd-timesyncd.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2021-09-06 06:02:08 UTC; 4h 30min ago
     Docs: man:systemd-timesyncd.service(8)
     Main PID: 398 (systemd-timesyn)
     Status: "Initial synchronization to time server 91.189.89.198:123 (ntp.ubuntu.com)."
     Tasks: 2 (limit: 41164)
     Memory: 1.6M
     CGroup: /system.slice/systemd-timesyncd.service
             └─398 /lib/systemd/systemd-timesyncd
```

Restart the service:

`$ sudo systemctl restart systemd-timesyncd.service`
