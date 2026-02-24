---
tool_name: cos
doc_type: troubleshoot
category: system
title: "SaaS API Monitoring"
summary: "Datadog Agent Collects HTTP endpoints. The HTTP check can detect bad response codes (e.g. 4**,5**), identify soon-to-expire SSL certificates, search responses for specific text,..."
---
# SaaS API Monitoring

Datadog Agent Collects HTTP endpoints. The HTTP check can detect bad response codes (e.g. 4**,5**), identify soon-to-expire SSL certificates, search responses for specific text, and much more. The check also submits HTTP response times as a metric.

We monitor HTTP (5**) Requests from SaaS

```bash
5xx Status Codes
    These are server errors. That means something went wrong with the response (website/server) and not the request (client/user). They include:
    •	500 – Internal Server Error
    •	502 – Bad Gateway
    •	503 – Service Unavailable
    •	504 – Gateway Timeout
```

### Step 1

Login in Datadog

### Step 2

Click on Dashboards

![](images/exaEdis_0-1637672170221.png)

### Step 3

Click on SaaS Prod Dashboard

![](images/exaEdis_1-1637672207532.png)

### Step 4

 Click on Web Services

 ![](images/exaEdis_0-1637743750474.png)

 ### Step 5

![](images/exaEdis_1-1637744781600.png)

```bash
HTTP 5** Requests Alert Alarm will be triggered if the threshold is 1 or above
```

### Step 5

Connect via AWS Console or CLI, follow the link: [this article](saas-login-to-sso.md)

### Step 6

First, check the datadog-agent status

`$ sudo datadog-agent status`

```sql
    openmetrics (1.14.0)
    --------------------
      Instance ID: openmetrics:api:c80a2220c63f96db [OK]
      Configuration Source: file:/etc/datadog-agent/conf.d/openmetrics.d/conf.yaml
      Total Runs: 148,905
      Metric Samples: Last Run: 30, Total: 3,608,699
      Events: Last Run: 0, Total: 0
      Service Checks: Last Run: 1, Total: 148,905
      Average Execution Time : 68ms
      Last Execution Date : 2021-09-08 11:44:10 UTC (1631101450000)
      Last Successful Execution Date : 2021-09-08 11:44:10 UTC (1631101450000)
```

If the Instance ID is OK then proceed to the next step, else restart the datadog-agent.

```bash
$ sudo restart datadog-agent
```

 If the error still occurs, please contact the Product Engineer Team

### Step 7

Filter the output with 5** HTTP Code:

```sql
$ curl http://localhost:9090/metrics | grep <HTTP CODE>
```

### Step 8

Create a SUPPORT ticket and collect logs, please follow the article: [article](how-to-get-log-files-from-exasol-saas-systems-temporary.md)

### Step 9

Find the PID of the WebUI-backend

```sql
$ ps -aux
root    1246631  0.1  0.2 750452 69172 ?  Ssl  Aug13  38:32 /usr/opt/EXASuite-7/EXAClusterOS-7.2.0-dev1/share/webui_backend/saas start --addr :4444 --rpc-endpoint https://localhost:20003 --signup-url https://<url>.e
```

### Step 10

Kill the process ID

`$ kill -9 PID`

```sql
Note: COS will restart the webui_backend
```

### Step 11

After the successful restart please double-check:

1. Web UI
2. Datadog Status
3. Dashboard
4. Curl the URL path

```sql
Please contact Product Engineer Team if the issue has not been resolved
```
