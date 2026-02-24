---
tool_name: confd_client
doc_type: troubleshoot
category: system
title: "Log collection 101"
summary: "The process of pulling logs from Exasol is quite straightforward. However, the processes around it are quite diverse."
---
# Log collection 101

## Overview

The process of pulling logs from Exasol is quite straightforward. However, the processes around it are quite diverse.

This article provides an overview of how one would tackle it.

## Basics

For deployments with EXAoperation logs are collected via tool called `get_support_info`. For all other types of deployments (pre-NGA, NGA, Docker, SaaS) one uses tool called `exasupport`.
Usually other application just call one of these two commands.

The process of transferring the logs from customer to Exasol premises is explained in [Transfer customer log files to developers support hosts](/Support-and-Services/transfer-customer-log-files-to-developers-support-hosts.md). It will work for the majority of the customers. For customers with special access please look for instruction in Confluence (space "Integrated Professional Services" / IPS Projects / 2. Customer Projects).

## Which log files are needed by R&D

Depending on the issue a customer has, R&D needs different type of log files.
Generally:

* in case of a SQL/ETL issue, SQL processes are needed
* in case of an issue with Backups, the DB itself server processes & backtraces are needed
* in case of an issue with COS, EXAClusterOS logs are needed
* only on currently ongoing problems, backtraces may be needed

Detailed in [Support scenarios and needed resources](https://exasol.atlassian.net/wiki/spaces/SPOTSUP/pages/4555971/Support+scenarios+and+needed+resources).

## Deployments with EXAoperation + we are responsible for pulling the logs

Small log sets could be pulled directly via EXAoperation according to [Log Files for Support, version 7.1](https://docs.exasol.com/db/7.1/administration/on-premise/support.htm) and it's subpages.

Single-session logs are always small. Server Processes logs are sometimes small. The rest is usually big.

For big log sets one would ssh into the cluster and run `get_support_info` there to generate the logs. More information on the process and on the content of log bundle could be found in [Exasol log files explained v7](Exasol-log-files-explained-v7.md).

## Deployments with EXAoperation + we are NOT responsible for pulling the logs

We instruct customer to pull the logs via EXAoperation according to a corresponding subpage of [Log Files for Support, version 7.1](https://docs.exasol.com/db/7.1/administration/on-premise/support.htm) (we give customer a particular link and tell to generate logs according to it). If no page corresponds exactly to the log set that we need, then we tell customer which options in EXAoperation to use. Then we instruct customer to upload the generated logs file via Log File Upload URL from Salesforce case.

## Deployments with v8

As of 2025-04-30 logs are generated exclusively via `exasupport` command.

So, depending on whose responsibility is to collect logs for a particular case, either we or customer should follow [Log Files for Support, version 8](https://docs.exasol.com/db/latest/administration/on-premise/support.htm) and it's subpages to generate the logs.  If no page corresponds exactly to the log set that we need, then we tell customer which `exasupport` options to use (or use them ourselves, respectively).The generated log file needs to be uploaded to Log File Upload URL from Salesforce case.

More information on the process and on the content of log bundle could be found in [Exasol log files explained v8](Exasol-log-files-explained-v8.md).

## SaaS Deployments

SaaS is running version 8, so logs are generated using `exasupport` tool.
However, the overall setup makes transferring the logs in a usual way hard, therefore one should follow the following article to pull the logs: [How to get log files from Exasol SaaS systems (temporary solution)](/SaaS/how-to-get-log-files-from-exasol-saas-systems-temporary.md).

If COS is not running, it is still possible to get the logs: [Collect logs from no-container running clusters/databases on SaaS](/SaaS/collect-logs-from-no-container-running-clusters-on-saas.md).

It is also good to know that SaaS rotates logs also to the S3 bucket that corresponds to the particular deployment (can be found in database CloudFormation stack resources by searching for S3Bucket), in folders `logs/[node name]`. Rotated logs are preserved here much longer than in COS.

## Pre-NGA and Docker Deployments

Logs are generated using `exasupport` tool.

So guidance for this tool from [Log Files for Support, version 8](https://docs.exasol.com/db/latest/administration/on-premise/support.htm) should generally work, as `exasupport` interface is so far (2025-04-30) backwards compatible.

What would be different - prior version 8 there was no c4, so one will be accessing container via ssh or Docker-specific commands.

The way to call `confd_client` slightly changed in version 8, so prior it `confd_client` would require `-c` before the command name and parameters in JSON form after `-a`, like

```shell
confd_client -c bucketfs_add -a '{"bucketfs_name":"newbucket-bucketfs", "http_port":"6932", "https_port":"6933", "owner":[500, 500]}'
```

Copying files for deployments prior to version 8 is also to be done via `scp` or `cp`

```shell
# scp using local path to file inside COS
scp -P 20002 root@localhost:/exa/tmp/support/[log file name].tar.gz /home/exasol

# cp using absolute path on host
cp -p /home/exasol/.ccc/play/local/*/main/*/data/tmp/support/[log file name].tar.gz /home/exasol/
```

## Additional References

* [Transfer customer log files to developers support hosts](/Support-and-Services/transfer-customer-log-files-to-developers-support-hosts.md)
* [Support scenarios and needed resources](https://exasol.atlassian.net/wiki/spaces/SPOTSUP/pages/4555971/Support+scenarios+and+needed+resources)
* [Log Files for Support, version 7.1](https://docs.exasol.com/db/7.1/administration/on-premise/support.htm)
* [Exasol log files explained v7](Exasol-log-files-explained-v7.md)
* [Log Files for Support, version 8](https://docs.exasol.com/db/latest/administration/on-premise/support.htm)
* [Exasol log files explained v8](Exasol-log-files-explained-v8.md)
* [How to get log files from Exasol SaaS systems (temporary solution)](/SaaS/how-to-get-log-files-from-exasol-saas-systems-temporary.md)
* [Collect logs from no-container running clusters/databases on SaaS](/SaaS/collect-logs-from-no-container-running-clusters-on-saas.md)


