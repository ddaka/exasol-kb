---
tool_name: confd_client
doc_type: troubleshoot
category: system
title: "SaaS Customer Database/Cluster Deployment Monitor"
summary: "When a customer creates a database or cluster, there will be created a new database/cluster deployment monitor. This monitor basically will monitor how long it's taking for this..."
---
# SaaS Customer Database/Cluster Deployment Monitor

## Overview

When a customer creates a database or cluster, there will be created a new database/cluster deployment monitor. This monitor basically will monitor how long it's taking for this database or cluster to be created. We have set a threshold of 20 minutes, so, if it will take 20 minutes or more, we will receive the alert. The alert for this monitor is sent to people that are on-call through OpsGenie, on mobile, and on slack channel.

The alert will contain all information needed such as account uuid, database uuid and cluster uuid of the database/cluster that is taking more 20 minutes to be created. These information are needed to continue troubleshooting.

## Prerequisites

* You must have access on SaaS Production and SaaS Production Customers AWS accounts
* You must have access on SaaS Management node and Customer Database nodes (Optional)
	+ Follow the steps in [this](saas-how-to-connect-to-the-instance.md) article on how to do that
* You must be part of SaaS Alerts channel in Slack or you must have access on Datadog Production Account
* You need to extract Account UUID, Database UUID and Cluster UUID from alert description

## Explanation

Once the customer has created the database or cluster, the database/cluster deployment monitor has been created in Datadog. This is how it looks in Datadog:

![](images/DeploymentMonitor.png)

As database/cluster continues its process to be created, this monitor will take a look on how long this database/cluster is taking to be created. Once it will pass the threshold, which is 20 minutes, then will trigger an alarm. Alarm then is sent to the on-call people and in Slack channel. This is how such alert looks in slack:

![](images/DeploymentMonitorSlack.png)

As you can see, alert will contain all needed information regarding Account UUID and DB UUID for database deployment monitor and Account UUID, DB UUID and Cluster UUID for cluster deployment monitor. Once we have all these information, we can continue our troubleshooting process.

## Troubleshooting

Some hints when troubleshooting database/cluster deployment monitor alerts.

* First job that will be executed in management node when customer creates a database is *saas_db_create*. This is a confd job and you can track its progress on /exa/logs/cored/confd.*.log file. Check if there any error on this job.
* First job that will be executed in management node when customer creates a cluster is *saas_cluster_create*. This is a confd job and you can track its progress on /exa/logs/cored/confd.*.log file. Check if there is any error on this job.
* Check if SaaS Engine has received the event in management node. You can check it in /exa/logs/cored/saasd.*.log. Check if there is any error processing this event. It will be looked something like:
	+ *Operation (creating, deployment_creating) received for cluster Cluster01, uuid: sJ8f5h2hTvudH0r5qTOJrg*
	+ or for cluster:
		- *Operation (creating, worker_cluster_creating) received for cluster Cluster_Playbook2, uuid: 5UPGJ6ddTkSPs2msB_2A9g*
* Outside of container, in the host machine of management node, check the c4 service logs. c4 is responsible to create the customer database cloudformation stack and other infrastructure resources. Check if there is any error on c4 logs by executing the following command in host machine:
	+ *sudo journalctl -u c4.service*
	+ and the log entry looks something like:
		- *Calling function: create_deployment with parameters map[account_uuid:701Awn1NRcCniNHzZ4FUCg client_id:0oa1a0sje5WuLz4oQ417 cluster_dnid:3o65mb5odjf6vfd54bfoq2jqca cluster_uuid:273WB64aS-qUfeBK6GkwEA db_name:273WB64aS-qUfeBK6GkwEA db_uuid:Qxj69I__QHyrUlN_DwbxEg instance_type:r5d.large num_nodes:4 platform:aws platform_reference: region:eu-central-1 size:XS working_copy:default]*
	+ while for cluster creation, the only log entry in c4 will look like:
		- *Calling function: create_dns_record with parameters map[cluster_dnid:4vb4mj5hlvhejd5tngwap7ma6y cluster_uuid:5UPGJ6ddTkSPs2msB_2A9g platform:aws region:eu-central-1]*
* Get database platform reference. Database platform reference is the name of CloudFormation stack name in SaaS Production Customers account. This id can be taken by executing following command:
	+ *confd_client -c saas_db_info -a '{"account_uuid":"701Awn1NRcCniNHzZ4FUCg", "db_uuid":"Qxj69I__QHyrUlN_DwbxEg"}'*
* Check if customer database CloudFormation stack has been created successfully in SaaS Production Customers account. Go to AWS Console -> CloudFormation (in search bar) -> paste database platform reference in stacks search bar
* Check if database/clusters instances are booted properly. Go to AWS Console -> EC2 (in search bar) -> paste DB or Cluster UUID in EC2 instances search bar. There EC2 will list all DB or Cluster instances. Go through each of them and check whether the c4 service has been started properly. You can connect to each of them using Session Manager (for more information, follow this [link](saas-how-to-connect-to-the-instance.md)). An instance is booted properly if c4 stage 6 is finished successfully. You can check the c4 logs by executing the following command:
	+ *sudo journalctl -u c4_cloud_command.service*
	+ Log entry for stage 6 looks like:
		- *stage6: All stages finished.*
* Inside of node 10 (the so-called Access Node), check whether the database status is *running* and not *setup*using the following command:
	+ *dwad_client list*
* When cluster creation has been triggered, inside of node 10, check whether the following confd jobs are finished successfully inside of /exa/logs/cored/confd.*.log:
	+ *infra_worker_db_add*
	+ *infra_instances_add*

## Logs

When troubleshooting session is finished, collect all the results and create a support ticket. If possible, collect logs for the database/cluster (for more information, check the [article](how-to-get-log-files-from-exasol-saas-systems-temporary.md)).

## Additional References

References to other database/cluster states playbooks:

[Starting monitor](saas-customer-database-cluster-starting-monitor.md)

[Deleting monitor](saas-customer-database-cluster-deleting-monitor.md)

[Stopping monitor](saas-customer-database-cluster-stopping-monitor.md)

[Error monitor](saas-customer-database-cluster-error-monitor.md)
