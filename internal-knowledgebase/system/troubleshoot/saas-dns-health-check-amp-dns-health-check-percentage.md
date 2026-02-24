---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "SaaS - DNS Health Check &amp; DNS Health Check Percentage"
summary: "In this article, we will try to cover every possibility as to why DNS might be failing.## Prerequisites"
---
# SaaS - DNS Health Check &amp; DNS Health Check Percentage

## Overview

In this article, we will try to cover every possibility as to why DNS might be failing.## Prerequisites

* Access to Datadog Production Account
* SaaS Production AWS Account

### Step 1

Click on Dashboards on left navigation menu:

![](images/exaEdis_0-1638272131652.png)

### Step 2

Click on ***SaaS PROD Dashboard***

### Step 3

Click on the ***Warning / Alert*** under the Management Services column

### Step 4

Under the list of Management Services monitors, click on ***PROD MGMT DNS Health Check*** monitor and check how long the DNS status is ***Warning / Alert / No Data***

### Step 5

Under the list of Management Services monitors, click on ***PROD MGMT DNS Health Check Percentage*** monitor and check how long the monitor is ***Warning / Alert / No Data***. The there is something wrong with DNS during last 5 minutes, and percentage is dropped below the given threshold 18%, it will trigger the alarm.

### Step 6

There may be more than one reason for having a DNS problem, and we will try to cover every possibility in the following sections:

### Option 1

Check instance state:

 **Step 1:**
Login to AWS Console

 **Step 2:**
Check the instance state

**Note:** If the instance is in other states than running, you must stop and start the instance. Please contact the Production Engineering Team and create a Support Ticket.

### Option 2

Check Target Group Status

 **Step 1:**

 Login to AWS Console

 **Step 2:**

 Go to EC2 -> Target Groups

**Step 3:**

Check the Health Status for the Port

**Step 4:**
If port health Status is Unhealthy, follow [this article](saas-api-monitoring.md) from Step 9:

**Note:** Please raise a Support ticket and contact the Production Engineering Team if the issue still exists.

### Option 3

Use dig to query nameservers

**Step 1:**

Check the Status of DNS
`$dig cloud.exasol.com`

**Note:**  Please contact the Production Engineering Team if the status is other than NOERROR

**Step 2:**

Get the list of nameservers
`$ dig NS +short cloud.exasol.com`

**Step 3:**

Compare with Route Recorded nameservers

- Login to SaaS Production AWS Account Console
- Go to Route 53 -> Hosted Zones
- Click on Domain NS Entry ex. *cloud.exasol.com - NS*
- Compare nameservers

**Note:** If NS doesn't match please contact Production Engineering Team.​
