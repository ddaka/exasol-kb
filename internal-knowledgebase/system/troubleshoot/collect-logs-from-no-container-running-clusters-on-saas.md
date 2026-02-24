---
tool_name: c4
doc_type: troubleshoot
category: system
title: "Collect logs from no-container running clusters/databases on SaaS"
summary: "There are cases where container on one or all instances belonging to a database or clusters might not be running or didn't start the proper way, therefore the well-known fully..."
---
# Collect logs from no-container running clusters/databases on SaaS

## Overview

There are cases where container on one or all instances belonging to a database or clusters might not be running or didn't start the proper way, therefore the well-known fully automated log collection (described [here](how-to-get-log-files-from-exasol-saas-systems-temporary.md)) way does not work. Or there are cases that instances itself don't boot properly and we must collect `c4_cloud_command` logs for further analysis. For such cases, there is no automated way of log collection yet, therefore the manual process of doing it is described here.

## Prerequisites

* Access to SaaS Production Customers AWS account
* Access to Bug Tracking Server - *submit01.dev.exasol.com*

## Procedure

Here we will describe all steps needed to collect logs on cluster/database instances where container is not running.

1. Go to AWS Console of SaaS Production Customers account and navigate to EC2
2. Filter out the instances by using cluster/database uuid

3. Create a session to the instance, either using CLI or directly on AWS Console (as described [here](saas-how-to-connect-to-the-instance.md))
4. If you want to collect c4_cloud_command logs:
	1. `sudo journalctl -u c4_cloud_command.service | tail -350 > n10-c4-cloud-command.log` -> 350 is the number of last lines, usually from the instance reboot and forward
	2. `aws s3 cp n10-c4-cloud-command.log s3://[BUCKET_NAME]/logs/SUPPORT_[ID]/` ->BUCKET_NAME can be found in database CloudFormation stack resources by searching for *S3Bucket*, [ID] is the number of the support ticket you are working on. ex: `aws s3 cp n10-c4-cloud-command.log s3://njdbmgso-s3bucket-xryaz1h4jx70/logs/SUPPORT-27520/`
5. If you want to get all container logs, without a container being up and running, you can collect them:
	1. `cd ~/.ccc/play/local/*/main/10/data/logs/cored` - change directory to the container logs directory
	2. `sudo tar -czf /tmp/n10-cos-logs.tar.gz .` - zip all log files
	3. `aws s3 cp /tmp/n10-cos-logs.tar.gz s3://[BUCKET_NAME]/logs/SUPPORT_[ID]/` - copy the zipped file to S3. BUCKET_NAME can be found in database CloudFormation stack resources by searching for S3Bucket, [ID] is the number of the support ticket you are working on. ex: `aws s3 cp /tmp/n10-cos-logs.tar.gz s3://njdbmgso-s3bucket-xryaz1h4jx70/logs/SUPPORT-27520/`
6. Go to Bug Track Server and create the folder for the ticket you are working on
	1. `cd /srv/bugtrack/SAAS/` - change the directory to the SaaS one
	2. `mkdir SUPPORT_[ID]` - [ID] is the number of the support ticket you are working on
	3. `cd SUPPORT_[ID]`
7. Login to AWS as described on Step 1 and Step 2 in this [playbook](how-to-get-log-files-from-exasol-saas-systems-temporary.md)
8. Once logged in, download the log files from S3
	1. `aws s3 cp s3://[BUCKET_NAME]/logs/SUPPORT_[ID]/n10-cos-logs.tar.gz .` or
	2. `aws s3 cp s3://[BUCKET_NAME]/logs/SUPPORT_[ID]/n10-c4-cloud-command.log .` -> download the file from S3
9. On every instance of cluster/database, repeat steps 3 - 8.

## Additional Notes

Once all the logs are downloaded, share the support ticket directory with developers by commenting on support ticket. The whole above process can and should be automated in the future.

## Additional References

AWS CLI S3 - <https://docs.aws.amazon.com/cli/latest/reference/s3/>
