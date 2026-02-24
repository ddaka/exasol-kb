---
tool_name: confd_client
doc_type: troubleshoot
category: system
title: "SaaS Customer Database/Cluster Deleting Monitor"
summary: "When a customer deletes a database or cluster, there will be a database/cluster deleting monitor that will track the database/cluster deletion. This monitor basically will monitor..."
---
# SaaS Customer Database/Cluster Deleting Monitor

## Overview

When a customer deletes a database or cluster, there will be a database/cluster deleting monitor that will track the database/cluster deletion. This monitor basically will monitor how long it's taking for this database or cluster to be deleted. We have set a threshold of 20 minutes, so, if it will take 20 minutes or more, we will receive the alert.

The alert for this monitor is sent to people that are on-call through OpsGenie, on mobile, and on slack channel. The alert will contain all information needed such as account uuid, database uuid and cluster uuid of the database/cluster that is taking more 20 minutes to be deleted. These information are needed to continue troubleshooting.

## Prerequisites

* You must have access on SaaS Production and SaaS Production Customers AWS accounts
* You must have access on SaaS Management node and Customer Database nodes (Optional)
	+ Follow the steps in [this](saas-how-to-connect-to-the-instance.md) article on how to do that
* You must be part of SaaS Alerts channel in Slack or you must have access on Datadog Production Account
* You need to extract Account UUID, Database UUID and Cluster UUID from alert description

## Explanation

Once the customer clicks on database/cluster delete  button and it starts deleting, the database/cluster deleting monitor starts counting the time:

![](images/DeletingMonitor.png)

As database/cluster continues its process to be deleted, this monitor will take a look on how long this database/cluster is taking to be deleted. Once it will pass the threshold, which is 20 minutes, then will trigger an alarm. Alarm then is sent to the on-call people and in Slack channel. This is how such alert looks in slack:

![](images/DeletingMonitorSlack.png)

As you can see, alert will contain all needed information regarding Account UUID and DB UUID for database deleting monitor and Account UUID, DB UUID and Cluster UUID for cluster deleting monitor. Once we have all these information, we can continue our troubleshooting process.

## Troubleshooting

Some hints when troubleshooting database/cluster deleting monitor alerts.

* First job that will be executed in management node when customer deletes a database is *saas_db_delete*. This is a confd job and you can track its progress on /exa/logs/cored/confd.*.log file. Check if there any error on this job.
* First job that will be executed in management node when customer deletes a cluster is *saas_cluster_delete*. This is a confd job and you can track its progress on /exa/logs/cored/confd.*.log file. Check if there is any error on this job.
* Check if SaaS Engine has received the event in management node. You can check it in /exa/logs/cored/saasd.*.log. Check if there is any error processing this event. It will be looked something like:
	+ *Operation (deleting, worker_cluster_deleting) received for cluster Cluster_Playbook2, uuid: 5UPGJ6ddTkSPs2msB_2A9g*
* Outside of container, in the host machine of management node, check the c4 service logs. c4 is responsible to delete the customer database cloudformation stack and other infrastructure resources. Check if there is any error on c4 logs by executing the following command in host machine:
	+ *sudo journalctl -u c4.service*
	+ and the log entry looks something like:
		- *Calling function: delete_deployment with parameters map[cluster_dnids:[takwshf47bgs3ary72zyjyxsau] cluster_uuids:[mBVpHLz4TS2COP6zhOLyBQ] platform:aws platform_reference:tjplblpj region:eu-central-1]*
	+ while for cluster deletion, the only log entry in c4 will look like:
		- *Calling function: delete_dns_record with parameters map[cluster_dnid:4vb4mj5hlvhejd5tngwap7ma6y cluster_uuid:5UPGJ6ddTkSPs2msB_2A9g platform:aws region:eu-central-1]*
* Get database platform reference. Database platform reference is the name of CloudFormation stack name in SaaS Production Customers account. This id can be taken by executing following command:
	+ *confd_client -c saas_db_info -a '{"account_uuid":"701Awn1NRcCniNHzZ4FUCg", "db_uuid":"Qxj69I__QHyrUlN_DwbxEg"}'*
* Check if customer database CloudFormation stack has been deleted successfully in SaaS Production Customers account. Go to AWS Console -> CloudFormation (in search bar) -> paste database platform reference in stacks search bar -> select Deleted from drop-down list -> if the stack status is different from DELETE_COMPLETE, something went wrong with c4 *delete_deployment* call.
* When cluster deletion has been triggered, inside of node 10, check whether the following confd jobs are finished successfully inside of /exa/logs/cored/confd.*.log:
	+ *infra_worker_db_remove*
	+ *infra_instances_remove*
* When cluster deletion has been triggered, check cluster instances are terminated in AWS. Go to AWS Console -> EC2 (in search bar) -> paste Cluster UUID in instances search bar -> if the instances are listed and their status is different than TERMINATED, something went wrong with below mentioned confd jobs

## Logs

When troubleshooting session is finished, collect all the results and create a support ticket. If possible, collect logs for the database/cluster (for more information, check the [article](how-to-get-log-files-from-exasol-saas-systems-temporary.md)).

## Additional References

References to other database/cluster states playbooks:

[Starting monitor](saas-customer-database-cluster-starting-monitor.md)

[Deployment monitor](saas-customer-database-cluster-deployment-monitor.md)

[Stopping monitor](saas-customer-database-cluster-stopping-monitor.md)

[Error monitor](saas-customer-database-cluster-error-monitor.md)
