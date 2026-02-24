---
tool_name: confd_client
doc_type: troubleshoot
category: system
title: "SaaS Customer Database/Cluster Error Monitor"
summary: "There might be different reasons why would a customer database or cluster go to *error* status. First of all, database/cluster might go to *error* status when one of four cluster..."
---
# SaaS Customer Database/Cluster Error Monitor

## Overview

There might be different reasons why would a customer database or cluster go to *error* status. First of all, database/cluster might go to *error* status when one of four cluster or database nodes get crashed or stopped directly through AWS Console or AWS API. Secondly, it might go to *error* status when there is a network issue and SaaS Engine can't contact database access node.

The error status means, SaaS Engine (in management node), is aware that there is something wrong with customer database/cluster and therefore updated database/cluster status in SaaS DB with *error* status. The *error* status usually comes after a continuous status of database/cluster such as *creating*, *starting* or *stopping*. It might be also be as result of job responsible to create/start/stop database/cluster timed out and therefore didn't finish successfully.

## Prerequisites

* You must have access on SaaS Production and SaaS Production Customers AWS accounts
* You must have access on SaaS Management node and Customer Database nodes (Optional)
	+ Follow the steps in [this](saas-how-to-connect-to-the-instance.md) article on how to do that
* You must be part of SaaS Alerts channel in Slack or you must have access on Datadog Production Account
* You need to extract Account UUID, Database UUID and Cluster UUID from alert description

## Explanation

*Error* status is a static status that is triggered just once. Everything can cause the *error* status for a database/cluster, therefore knowing the previous status of database/cluster might help a lot. *Error* monitor will trigger an alarm as soon as it is received in Datadog. This is how it looks in Datadog:

![](images/ErrorMonitor.png)

As database/cluster switches in error status, plugin will send the error status to Datadog immediately and the current threshold for such status is 1, therefore it means that once it receives the error status, the monitor will trigger the alarm immediately. Alarm then is sent to the on-call people and in Slack channel. This is how such alert looks in slack:

![](images/ErrorMonitorSlack.png)

As you can see, alert will contain all needed information regarding Account UUID and DB UUID for database starting monitor and Account UUID, DB UUID and Cluster UUID for cluster starting monitor. Once we have all these information, we can continue our troubleshooting process.

## Troubleshooting

Some hints when troubleshooting database/cluster error monitor alerts.

* Check what was the previous status of database/cluster before switching to error status. This can be done in SaaS DB.
* Based on previous status of database, check whether the respective confd jobs such as *saas_db_create*, *saas_db_start*, *saas_db_stop* or *saas_db_delete* in management node are finished successfully. These are confd jobs and you can track their progress on /exa/logs/cored/confd.*.log file. Check if there any error on these jobs.
* Based on previous status of database, check whether the respective confd jobs such as *saas_cluster_create*, *saas_cluster_start,* *saas_cluster_stop* or *saas_cluster_delete* in management node are finished successfully. These are confd jobs and you can track their progress on /exa/logs/cored/confd.*.log file. Check if there any error on these jobs.
* Check if SaaS Engine has received the respective events in management node for the previous status of database/cluster such as *deployment_creating*, *main_cluster_starting*, *main_cluster_stopping, worker_cluster_creating, worker_cluster_starting, worker_cluster_stopping* or *worker_cluster_deleting*. You can check it in /exa/logs/cored/saasd.*.log. Check if there is any error while processing these events.
* If previous status of database was *deleting*, then get database platform reference. Database platform reference is the name of CloudFormation stack name in SaaS Production Customers account. This platform reference can be taken by executing following command:
	+ *confd_client -c saas_db_info -a '{"account_uuid":"701Awn1NRcCniNHzZ4FUCg", "db_uuid":"Qxj69I__QHyrUlN_DwbxEg"}'*
	+ then check whether the CloudFormation stack in SaaS Production Customer account is deleted or not.
		- you confirm that by going to SaaS Production Customers AWS account -> CloudFormation (in search bar) -> paste the database platform reference -> select Deleted from the drop-down list -> check whether its status is DELETE_COMPLETE
	+ check whether c4 call in management node for deleting customer database CloudFormation stack has been finished successfully, using the following command:
		- *sudo journalctl -u c4.service*
		- the output should look like:
		- *Calling function: delete_dns_record with parameters map[cluster_dnid:4vb4mj5hlvhejd5tngwap7ma6y cluster_uuid:5UPGJ6ddTkSPs2msB_2A9g platform:aws region:eu-central-1]*
* If previous status of database was *starting*, check whether all database/cluster instances are started properly. Make sure all stages of the *c4_cloud_command* inside of the instances are finished successfully, using following command:
	+ *sudo journalctl -u c4_cloud_command.service*
	+ and the output for a successful instance startup is:
		- *stage6: All stages finished.*
* Check also confd logs inside of n10 whether the starting related jobs such as *infra_db_start* and *infra_instances_start* are finished successfully.
	+ Go to SaaS Production Customers AWS account -> EC2 (in search bar) -> filter instances by database/cluster UUID and n10 -> create a session to the container inside instance -> check confd logs (usually /exa/logs/cored/confd.*.log)
	+ Also check the status of database(s) inside of n10 using *dwad_client list*
* If previous status of database was *stopping*, check whether all database/cluster instances are stopped. Check also confd logs inside of n10 whether the stopping related jobs such as infra_db_stop and *infra_instances_stop* are finished successfully.
* If previous status of database was *creating*, then get database platform reference. Database platform reference is the name of CloudFormation stack name in SaaS Production Customers account. This id can be taken by executing following command:
	+ *confd_client -c saas_db_info -a '{"account_uuid":"701Awn1NRcCniNHzZ4FUCg", "db_uuid":"Qxj69I__QHyrUlN_DwbxEg"}'*
	+ then check whether the CloudFormation stack in SaaS Production Customer account is created properly. This can be done by checking database stack status is in either CREATE_COMPLETE or UPDATE_COMPLETE
	+ check whether c4 call in management node for creating customer database CloudFormation stack has been finished successfully

## Logs

When troubleshooting session is finished, collect all the results and create a support ticket. If possible, collect logs for the database/cluster (for more information, check the [article](how-to-get-log-files-from-exasol-saas-systems-temporary.md)).

## Additional References

[Starting monitor](saas-customer-database-cluster-starting-monitor.md)

[Deployment monitor](saas-customer-database-cluster-deployment-monitor.md)

[Deleting monitor](saas-customer-database-cluster-deleting-monitor.md)

[Stopping monitor](saas-customer-database-cluster-stopping-monitor.md)
