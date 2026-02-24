---
tool_name: confd_client
doc_type: troubleshoot
category: system
title: "SaaS Customer Database/Cluster Stopping Monitor"
summary: "When a customer stops a database or cluster, there will be a database/cluster stopping monitor that will track the database/cluster stopping. This monitor basically will monitor..."
---
# SaaS Customer Database/Cluster Stopping Monitor

## Overview

When a customer stops a database or cluster, there will be a database/cluster stopping monitor that will track the database/cluster stopping. This monitor basically will monitor how long it's taking for this database or cluster to be stopped. We have set a threshold of 20 minutes, so, if it will take 20 minutes or more, we will receive the alert.

The alert for this monitor is sent to people that are on-call through OpsGenie, on mobile, and on slack channel. The alert will contain all information needed such as account uuid, database uuid and cluster uuid of the database/cluster that is taking more 20 minutes to be stopped. These information are needed to continue troubleshooting.

## Prerequisites

* You must have access on SaaS Production and SaaS Production Customers AWS accounts
* You must have access on SaaS Management node and Customer Database nodes (Optional)
	+ Follow the steps in [this](saas-how-to-connect-to-the-instance.md) article on how to do that
* You must be part of SaaS Alerts channel in Slack or you must have access on Datadog Production Account
* You need to extract Account UUID, Database UUID and Cluster UUID from alert description

## Explanation

Once the customer clicks on database/cluster stop button and it starts stopping, the database/cluster stopping monitor starts counting the time:

![](images/StoppingMonitor.png)

As database/cluster continues its process to be stopped, this monitor will take a look on how long this database/cluster is taking to be stopped. Once it will pass the threshold, which is 20 minutes, then will trigger an alarm. Alarm then is sent to the on-call people and in Slack channel. This is how such alert looks in slack:

![](images/StoppingMonitorSlack.png)

As you can see, alert will contain all needed information regarding Account UUID and DB UUID for database stopping monitor and Account UUID, DB UUID and Cluster UUID for cluster stopping monitor. Once we have all these information, we can continue our troubleshooting process.

## Troubleshooting

Some hints when troubleshooting database/cluster stopping monitor alerts.

* First job that will be executed in management node when customer deletes a database is *saas_db_stop*. This is a confd job and you can track its progress on /exa/logs/cored/confd.*.log file. Check if there any error on this job.
* First job that will be executed in management node when customer deletes a cluster is *saas_cluster_stop*. This is a confd job and you can track its progress on /exa/logs/cored/confd.*.log file. Check if there is any error on this job.
* Check if SaaS Engine has received the event in management node. You can check it in /exa/logs/cored/saasd.*.log. Check if there is any error processing this event. It will be looked something like:
	+ *Operation (stopping, main_cluster_stopping) received for cluster Cluster01, uuid: fZ4HXH0TSTu9yVsIB7Y-LQ*
	+ followed with:
		- *Executed job with id '10.2036' and payload '{'method': 'job_start', 'job': 'infra_db_stop', 'params': {'params': {'db_name': 'fZ4HXH0TSTu9yVsIB7Y-LQ', 'timeout': 600}}}'*
* When database/cluster stopping has been triggered, inside of node 10, check whether the following confd jobs are finished successfully inside of /exa/logs/cored/confd.*.log:
	+ *infra_db_stop*
	+ *infra_instances_stop*
* Check if customer database/cluster instances have been stopped successfully in SaaS Production Customers account. Go to AWS Console -> EC2 (in search bar) -> paste database or cluster UUID in instances search bar -> if any nodes of a cluster or database (except of n10 - Access Node) is in a status different that stopped, check the logs in n10 for the above mentioned jobs.

## Logs

When troubleshooting session is finished, collect all the results and create a support ticket. If possible, collect logs for the database/cluster (for more information, check the [article](how-to-get-log-files-from-exasol-saas-systems-temporary.md)).

## Additional References

[Starting monitor](saas-customer-database-cluster-starting-monitor.md)

[Deployment monitor](saas-customer-database-cluster-deployment-monitor.md)

[Deleting monitor](saas-customer-database-cluster-deleting-monitor.md)

[Error monitor](saas-customer-database-cluster-error-monitor.md)
