---
tool_name: confd_client
doc_type: troubleshoot
category: system
title: "SaaS Customer Database/Cluster Availability Monitor"
summary: "Database/cluster availability is checked by monitoring plugin which executes a select 1 query in every 1 minute if database connection is possible. If connection does not work it..."
---
# SaaS Customer Database/Cluster Availability Monitor

## Overview

Database/cluster availability is checked by monitoring plugin which executes a select 1 query in every 1 minute if database connection is possible. If connection does not work it will send alert to the Datadog. From metric point of view, 1 means connection database/cluster is connectable and 0 means, connection was not successful.

If db connection fails it could be various reasons, for example changing instances security group rules, database/cluster might have internal issues, networking changes etc.

## Prerequisites

* You must have access on SaaS Production and SaaS Production Customers AWS accounts
* You must have access on SaaS Management node and Customer Database nodes
	+ Follow the steps in [this](saas-how-to-connect-to-the-instance.md) article on how to do that
* You must be part of Slack channels (saas-alerts or opsgenie-alerts) or you are in on-call duty to receive alert
* You need to extract Account UUID, Database UUID and Cluster UUID from alert description
* You have access to Datadog Production Environment to check monitors & metrics (optional)

## Explanation

Once the customer has created the database or cluster, availability monitor has been created in Datadog. This is how it looks in Datadog:

![](images/Availability_Monitor.png)

As database/cluster keeps being in running state, monitoring plugin will constantly check whether database/cluster is connectable, and does that every minute. If for mentioned above reasons, database/cluster becomes un-connectable, and keeps being un-connectable for last 5 minutes, the monitor will trigger an alarm, and it will look in Slack as below:

![](images/Availability_Alert_Slack.png)

As you can see, the alert contains all needed information such as Account UUID, DB UUID and Cluster UUID to continue troubleshooting.

## Troubleshooting

Some hints when troubleshooting database/cluster availability monitor alerts.

* First thing we need to check is whether database/cluster instances are up and running. To do so, follow below steps:
	+ Go to SaaS Production Customers AWS account
	+ Search for EC2 service and open EC2 console
	+ Filter instances by Cluster UUID instance tag
	+ Make sure all instances are up/running and their instance checks have 2/2 checks passed
* Next, check whether database/cluster is up and running:
	+ Go to SaaS Production Customers AWS account
	+ Search for EC2 service and open EC2 console
	+ Filter instances by Cluster UUID instance tag
	+ Check `c4_cloud_command` state `systemctl status c4_cloud_command` and log files `sudo journalctl -fu c4_cloud_command`
	+ Connect to one of the instances and ssh to container (for more info follow steps in this [link](saas-how-to-connect-to-the-container.md))
	+ Execute *dwad_client list* command
	+ In the list of systems, search for the one by Name: &lt;Cluster UUID&gt;
	+ Check whether its state and connection state are running and up respectively

	![](images/exaFlakron_0-1643234338145.png)

	+ Check if there is any node has any problem, such as disk is full (`df -h`)
	+ Check uptime of all nodes to see if there was any restart, for example, for main cluster you can use this one liner: `for i in {11..14}; do ssh n$i uptime; done`
	+ Check if maximum number of concurrent session is reached (Check connection server logs `cd /exa/logs/db/$(dwad_client shortlist)` and `tail -f *_ConnectionServer_*`
	+ Restart databases, stop (`dwad_client stop-wait $(dwad_client shortlist)`) and start (`dwad_client start-wait$(dwad_client shortlist`)
	+ If there is no instance issue and if there is no obvious reason please collect logs and create a support ticket (for more info follow steps in this [link](how-to-get-log-files-from-exasol-saas-systems-temporary.md))
* If you are unable to ssh to the container in database/cluster instances due to c4 or c4_cloud_command issues, please follow this [link](collect-logs-from-no-container-running-clusters-on-saas.md) to collect logs when container is not running in customer database/cluster
* Next, check whether security groups inbound rules are in place for the SaaS Engine
	+ Go to SaaS Production Customers AWS account
	+ Search for EC2 service and open EC2 console
	+ Filter instances by Cluster UUID instance tag
	+ Click on one of the instances and navigate to Security tab
	+ Under Security groups, click on the attached security group
	+ By default, it will show inbound rules
	+ Check all the given below ports are open for SaaS Engine IP Range (usually 192.168.1.0/24)
		- It will look something like this:

		![](images/exaFlakron_0-1643234969421.png)

* In management node, check the confd logs:
	+ Go to SaaS Production AWS account
	+ Search for EC2 service and open EC2 console
	+ Filter instances by *management* instance tag value
	+ Connect to one of the instances and ssh to container (for more info follow steps in this [link](saas-how-to-connect-to-the-container.md))
	+ Open confd log file with *less* or *more* command (usually under */exa/logs/cored/confd.325.19.0.551.log* )
	+ Search for *Database is not connectable: &lt;Cluster UUID&gt;*
	+ You may find useful information above this line such as:
		- *Connection to db timed out!* or
		- *User password not found!*
	+ In such case, report this to ***Production Engineering Team***

## Logs

When troubleshooting session is finished, you may find it useful to collect logs from management node or from database/cluster. To do so, please check this [article](how-to-get-log-files-from-exasol-saas-systems-temporary.md).

## Additional References

References to database/cluster error state playbook:

[Error monitor](saas-customer-database-cluster-error-monitor.md)
