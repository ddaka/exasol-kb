---
tool_name: confd_client
doc_type: troubleshoot
category: system
title: "Change Monitors Threshold on SaaS Monitoring Plugin"
summary: "In order to increase or decrease the threshold of database/cluster monitors, we should do that on SaaS Monitoring plugin. There are two types of monitors: state and infrastructure..."
---
# Change Monitors Threshold on SaaS Monitoring Plugin

## Overview

In order to increase or decrease the threshold of database/cluster monitors, we should do that on SaaS Monitoring plugin. There are two types of monitors: state and infrastructure monitors. The thresholds differ from monitor to another, and we will need by time to time to change the threshold value.

## Prerequisites

* Access to Datadog Production Account
* SaaS Production AWS Account

## Explanation

As the SaaS product evolves, so does its monitoring plugin. To adjust databases/clusters monitors thresholds, we must go to SaaS management node and edit them on monitoring plugin. As we have two types of monitors, state and infrastructure monitors, the way to change the thresholds on state and infrastructure monitors differs. State monitors, which track the database/cluster states such as creating, starting, stopping, deleting and error, while infrastructure monitors track the underlying database/cluster resources such as CPU, Memory utilization, Disks, etc.

## Procedure

### Step 1

Connect to container inside of management node (follow [these steps](saas-how-to-connect-to-the-container.md) for more information)

### Step 2

Go to plugin directory inside bucketfs (usually under `/exa/data/bucketfs/bfsdefault/default` directory)

### Step 3

### State Monitors

 ### Step 1

 Open plugin configuration file (`plugin.config`) in write mode - usually `vi plugins\:plugin.config`

 ### Step 2

 Update cluster/database state monitors thresholds. For each state, there are two parameters:
 - Evaluation time `[STATE]_LAST_X` - for how many minutes X the monitor should track the state before reaching threshold
- Threshold `[STATE]_THRESHOLD` - the value in minutes which should trigger alerts

```
Note: Please keep in mind that some data points are sent to Datadog every 5 minutes and some are sent every 1 minute.
```

![](images/StateThresholds.png)

 ### Step 3

 Please note that restarting confd will restart other SaaS services as well. It will create a service interruption for customers for short time.

 Please follow these steps in this [article](saas-restart-confd-service.md) to restart ConfD service.

### Infrastructure Monitors

 ## Infrastructure Monitors:

 ### Step 1

 Open infrastructure monitors python file (`infra_monitors.py`) - usually `vi plugins\:infra_monitors.py`

 ### Step 2

 Find infrastructure monitor that you want to change its threshold. For ex. CPU monitor looks like:

![](images/InfraThresholds.png)

 ### Step 3

Inside of monitor properties, change its query by using new threshold value and evaluation time. Usually, for query alert monitors type, threshold is placed in end of query after the operator, while evaluation time is usually placed in parentheses after the function, which might be one of *avg, min, max, sum,* etc. For service check monitors type, threshold is placed only on parentheses after the function, such as *last(2)*. This is the number of minutes the monitor tracks if the service is not sending data. Once monitor does not receive data, in that case, in last 2 minutes, it will create an alert and send notification.

 ### Step 4

 Please note that restarting confd will restart other saas services as well. It will create a service interruption for customers for short time.

 Please follow these steps in this [article](saas-restart-confd-service.md) to restart ConfD service.

### Step 4

Changes will be in effect only for the future databases/clusters that are going to be created, but not for previous ones. In order to sync old monitors with new changes, sync module is needed to be executed. While you are in plugin directory, execute below command:

`python3 plugins\:sync.py`

## Additional References

Datadog Service Check Monitors - <https://docs.datadoghq.com/monitors/create/types/host>

Datadog Query Alert Monitors - <https://docs.datadoghq.com/monitors/create/types/metric>

Datadog State Monitors - search for "SaaS/Playbook - Customer Database/Cluster".
