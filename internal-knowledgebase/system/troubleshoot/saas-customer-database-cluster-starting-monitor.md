---
tool_name: confd_client
doc_type: troubleshoot
category: system
title: "SaaS Customer Database/Cluster Starting Monitor"
summary: "When a customer starts a database or cluster, there will be a database/cluster starting monitor that will track the database/cluster starting process. This monitor basically will..."
---
# SaaS Customer Database/Cluster Starting Monitor

## Overview

When a customer starts a database or cluster, there will be a database/cluster starting monitor that will track the database/cluster starting process. This monitor basically will monitor how long it's taking for this database or cluster to be started. We have set a threshold of 20 minutes, so, if it will take 20 minutes or more, we will receive the alert.

The alert for this monitor is sent to people that are on-call through OpsGenie, on mobile, and on slack channel. The alert will contain all information needed such as account uuid, database uuid and cluster uuid of the database/cluster that is taking more 20 minutes to be stopped. These information are needed to continue troubleshooting.

## Prerequisites

* You must have access on SaaS Production and SaaS Production Customers AWS accounts
* You must have access on SaaS Management node and Customer Database nodes (Optional)
	+ Follow the steps in [this](saas-how-to-connect-to-the-instance.md) article on how to do that
* You must be part of SaaS Alerts channel in Slack or you must have access on Datadog Production Account
* You need to extract Account UUID, Database UUID and Cluster UUID from alert description

## Explanation

Once the customer clicks on database/cluster start button, the database/cluster starting monitor starts counting the time:

![](images/StartingMonitor.png)

As database/cluster continues its process to be started, this monitor will take a look on how long this database/cluster is taking to be started. Once it will pass the threshold, which is 20 minutes, then will trigger an alarm. Alarm then is sent to the on-call people and in Slack channel. This is how such alert looks in slack:

![](images/StartingMonitorSlack.png)

As you can see, alert will contain all needed information regarding Account UUID and DB UUID for database stopping monitor and Account UUID, DB UUID and Cluster UUID for cluster stopping monitor. Once we have all these information, we can continue our troubleshooting process.

## Troubleshooting

Some hints when troubleshooting database/cluster starting monitor alerts.

* First job that will be executed in management node when customer deletes a database is *saas_db_start*. This is a confd job and you can track its progress on /exa/logs/cored/confd.*.log file. Check if there any error on this job.
* First job that will be executed in management node when customer deletes a cluster is *saas_cluster_start*. This is a confd job and you can track its progress on /exa/logs/cored/confd.*.log file. Check if there is any error on this job.
* Check if SaaS Engine has received the event in management node. You can check it in /exa/logs/cored/saasd.*.log. Check if there is any error processing this event. It will be looked something like:
	+ *Operation (starting, main_cluster_starting) received for cluster Cluster01, uuid: fZ4HXH0TSTu9yVsIB7Y-LQ*
	+ followed with:
		- *Executed job with id '10.3057' and payload '{'method': 'job_start', 'job': 'infra_db_start', 'params': {'params': {'db_name': '273WB64aS-qUfeBK6GkwEA', 'timeout': 600}}}'*
	+ or for cluster:
		- *Operation (starting, worker_cluster_starting) received for cluster Cluster02-cs, uuid: sr8iYYQGQ3CnARXQwoc6xQ*
			* followed with:
				+ *Executed job with id '10.41843' and payload '{'method': 'job_start', 'job': 'infra_db_start', 'params': {'params': {'db_name': 'sr8iYYQGQ3CnARXQwoc6xQ', 'timeout': 600}}}'*
* When database/cluster starting has been triggered, inside of node 10, check whether the following confd jobs are finished successfully inside of /exa/logs/cored/confd.*.log:
	+ *infra_db_start*
	+ *infra_instances_start*
* Check if customer database/cluster instances have been started successfully in SaaS Production Customers account. Go to AWS Console -> EC2 (in search bar) -> paste database or cluster UUID in instances search bar -> if any nodes of a cluster or database (except of n10 - Access Node) is still in status stopped, check the logs in n10 for the above mentioned jobs.
* In every customer database/cluster instances, check whether all six stages of *c4_cloud_command* service have been successfully finished by executing the following command in host machine:
	+ *sudo journalctl -u c4_cloud_command.service*
	+ Log entry for stage 6 looks like:
		- *stage6: All stages finished.*
* Inside of node 10 (the so-called Access Node), check whether the database status is*running*and not*setup*using the following command:
	+ *dwad_client list*

## Logs

When troubleshooting session is finished, collect all the results and create a support ticket. If possible, collect logs for the database/cluster (for more information, check the [article](how-to-get-log-files-from-exasol-saas-systems-temporary.md)).

## Additional References

References to other database/cluster states playbooks:

[Deployment monitor](saas-customer-database-cluster-deployment-monitor.md)

[Deleting monitor](saas-customer-database-cluster-deleting-monitor.md)

[Stopping monitor](saas-customer-database-cluster-stopping-monitor.md)

[Error monitor](saas-customer-database-cluster-error-monitor.md)
