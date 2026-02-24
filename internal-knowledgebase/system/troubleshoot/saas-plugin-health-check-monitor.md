---
tool_name: confd_client
doc_type: troubleshoot
category: system
title: "SaaS - Plugin Health Check Monitor"
summary: "Monitoring Plugin Health Check will send data points to Datadog every minute, and is depended on confd service. Once plugin is added to confd, it will start to receive events from..."
---
# SaaS - Plugin Health Check Monitor

## Overview

Monitoring Plugin Health Check will send data points to Datadog every minute, and is depended on confd service. Once plugin is added to confd, it will start to receive events from Saasd, Confd and Healthd services. Healthd service events are received every minute. Based on healthd events, every minute, plugin will send data points to Datadog.

As long as Confd service is up and running, plugin is expected to send data to Datadog. Besides Confd service, plugin is depended on Datadog API and APP. API and APP keys must be valid in order for the plugin to send data to Datadog. If plugin fails to send data points to Datadog, the monitor will switch to NO_DATA state and will send the alert. Alert is sent to on-call people and Slack channel. As long as Plugin Health Check monitor is in NO_DATA state, plugin is consider down, therefore there will be no functionality.

## Prerequisites

* You must have access on SaaS Production
* You must be part of SaaS Alerts channel in Slack or you must have access on Datadog Production Account

## Explanation

Once SaaS management deployment is done, plugin will be installed and configured automatically. During plugin installation, plugin health chech monitor will be created in Datadog. This monitor will check whether monitoring plugin is sending data to Datadog and working as expected. This is how it looks in Datadog:

![](images/PluginHealthCheckMonitor.png)

While SaaS continues to operate, the plugin health check monitor will track whether it is receiving data to Datadog or not. If there is something wrong with confd service in SaaS management node or Datadog API and APP keys are not valid anymore, the monitor will switch to NO_DATA and will stop functioning. Once it is in NO_DATA, it will trigger an alarm and send it to the on-call people and in Slack channel. This is how such alert looks in slack:

![](images/PluginMonitor.png)

Alert contains information that the monitoring plugin in production environment is not sending data to Datadog.

## Troubleshooting

Some hints when troubleshooting plugin health check monitor alerts.

* Check SaaS management instance is up and running in AWS. Go to AWS SaaS Production account -> EC2 (in search bar) -> search for the instance using saas:BelongsTo tag = prod.cloud.exasol.com -> make sure instance status is running.
* Check the container inside of management node is up and running. Try connecting to it by creating a session. For more info, follow this [article](saas-how-to-connect-to-the-container.md).
* Inside of container check the confd service is up and running by executing the following command:
	+ *cosps -N | grep confd*
	+ It should show something like:
		- `[root@n11 ~]# cosps -N | grep confd`
		- `19     0     0     0     RA-I     11     -     confd`
* Inside of container check the saasd service is up and running by executing the following command:
	+ *cosps -N | grep saasd*
	+ It should show something like:
		- `[root@n11 ~]# cosps -N | grep saasd`
		- `32     0     0     0     RA-I     11     -     saasd`
* Inside of container check the healthd service is up and running by executing the following command:
	+ *cosps -N | grep healthd*
	+ It should show something like:
		- `[root@n11 ~]# cosps -N | grep healthd`
		- `14     0     0     0     RA-I     11     -     healthd`
* Check whether plugin is installed by looking in EXAConf, usually stored in */exa/etc/EXAConf*
* Inside of plugin directory in container, which usually is */exa/data/bucketfs/bfsdefault/default,* get Datadog API and APP keys from plugin config file by executing following command:
	+ *grep -i "api_key\|app_key" plugins\:plugin.config*
* Test Datadog is reachable by using the following command:
	+ `curl -X GET "<https://api.datadoghq.eu/api/v1/monitor?monitor_tags=saas_monitortype:plugin_monitor,saas_belongsto:prod.cloud.exasol.com>" \`
	+ `-H "Content-Type: application/json" \`
	+ `-H "DD-API-KEY: API_KEY" \`
	+ `-H "DD-APPLICATION-KEY: APP_KEY"`
	+ If there is a forbidden response, API or APP keys is invalid
