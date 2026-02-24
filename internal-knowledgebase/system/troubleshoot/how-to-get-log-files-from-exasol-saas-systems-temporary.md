---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "How to get log files from Exasol SaaS systems (temporary solution)"
summary: "This script is a temporary solution until the full implementation of the \"R&D access to SaaS logs\" solution is ready."
---
# How to get log files from Exasol SaaS systems (temporary solution)

## Overview

This script is a temporary solution until the full implementation of the "R&D access to SaaS logs" solution is ready.

Details: [SAAS-887: Create temporary solution until full implementation is ready](https://exasol.atlassian.net/browse/SAAS-887)

In this article, you can find how to get log files and upload them to the **submit01** server from Exasol SaaS systems.

## Prerequisites

* The `exasol-log-collection` SSM document needs to be created beforehand. This is required to be done once and the Production Engineering team is responsible for this. Details: <https://github.exasol.com/saas/saas-infra/blob/master/ssm-documents/documents/app.py>
* Access to `submit01.dev.exasol.com` server
* Access to AWS SaaS Management and Customer account. Both of these accounts need to be configured in `aws cli` setting in **submit01** server.

## How to configure AWS SSO with AWS CLI in submit01 server

If AWS CLI is not configured, please configure it in order to have access to management and customer accounts. Once this is done next time you can just use `aws sso login --profile [Your AWS Profile Name]` command.

Example configuration:

### Step 1: Set proxy settings for submit01.dev.exasol.com

```shell
repoproxy
```

### Step 2: Configure AWS SSO

```bash
$ BROWSER=true aws configure sso

SSO start URL [None]: https://exasol-ct.awsapps.com/start#/
SSO Region [None]:  us-east-1
Attempting to automatically open the SSO authorization page in  your default browser.
If the browser does not open or you wish to use a different   device to authorize this request, open the following URL:
https://device.sso.us-east-1.amazonaws.com/
Then enter the code:
GMKP-SFRQ
There are 18 AWS accounts available to you.
Using the account ID 533884745449
The only role available to you is: AWSAdministratorAccess
Using the role name "AWSAdministratorAccess"
CLI default client Region [None]:   eu-central-1
CLI default output format [None]:   Json
CLI profile name [AWSAdministratorAccess-533884745449]:   dev-saas
To use this profile, specify the profile name using --profile,  as shown:
aws s3 ls --profile dev-saas
```

## How to get log files from the SaaS database (SaaSDB)

### Step 1: Get required arguments for `collect-logs.py` script

The `collect-logs.py` script is deployed in `/opt/log-management/temporary-solution` folder.

In order to get all required arguments and detailed information you can use help:

```bash
$ python3 collect-logs.py --help

usage: log_collection_client [-h] --db {CustomerDB,SaaSDB} [--mgmt-profile MGMT_PROFILE] [--customer-profile CUSTOMER_PROFILE] [--saas-region SAAS_REGION]
                             [--customer-db-region CUSTOMER_DB_REGION] [--saas-url SAAS_URL] [--db-uuid DB_UUID] --log-prefix LOG_PREFIX --start-time START_TIME
                             --stop-time STOP_TIME

Tool for collecting log files from Exasol SaaS

Arguments:
  -h, --help            show this help message and exit

Log Collection Config:
  --db {CustomerDB,SaaSDB}
                        Get logs from specified database. Should be set SaaSDB or CustomerDB
  --mgmt-profile MGMT_PROFILE
                        AWS profile name for Management Account. This argument required if --db argument set as 'SaaSDB'
  --customer-profile CUSTOMER_PROFILE
                        AWS profile name for Customer Account. This argument required if --db argument set as 'CustomerDB'
  --saas-region SAAS_REGION
                        AWS region where SaaS deployed. by default it is set as 'eu-central-1'
  --customer-db-region CUSTOMER_DB_REGION
                        AWS region where Customer database is running. This argument required if --db argument set as 'CustomerDB'
  --saas-url SAAS_URL   The URL of SaaS (ex: dev.dev-saas.exasol.com) This argument required if --db argument set as 'SaaSDB'
  --db-uuid DB_UUID     Customer Database UUID. This argument required if --db argument set as 'CustomerDB'
  --cluster-uuid CLUSTER_UUID
                        Customer Database Cluster UUID. This argument required if --db argument set as 'CustomerDB'
  --log-prefix LOG_PREFIX
                        Prefix for logs. This prefix will be used as a prefix for logs in S3 bucket
  --debug-info DEBUG_INFO
                        Debuginfo to retrieve, separated by comma (1 = EXAClusterOS logs, 2 = Coredumps, 3 = EXAStorage metadata or 0
                        for all)
  --exasolution-log-type EXASOLUTION_LOG_TYPE
                        EXASolution log type, separated by comma (1 = All, 2 = SQL processes, 3 = Server processes)
  --start-time START_TIME
                        Start time of logs (YYYY-MM-DD [HH:MM])
  --stop-time STOP_TIME
                        Stop time of logs (YYYY-MM-DD [HH:MM])

Step 2: Execute
```

### Step 2: Execute `collect-logs.py` with required arguments for downloading log files from `SaaSDB`

```bash
$ python3.6 /opt/log-management/collect-logs.py --db SaaSDB --mgmt-profile prod-saas --saas-region eu-central-1 --saas
-url prod.cloud.exasol.com --log-prefix Support-00001 --debug-info 1 --exasolution-log-type 3 --start-time 2021-09-30 --stop-time 2021-09-30

Directory '/srv/bugtrack/SAAS/Support-00001-5171087694/' created.
Starting to download log file 'exacluster_debuginfo_2021_09_30-10_40_32.tar.gz' from s3...
All log files downloaded to '/srv/bugtrack/SAAS/Support-00001-5171087694/' directory.
```

## How to get log files from the Customer SaaS database (CustomerDB)

### Step 1: Get the required arguments for `collect-logs.py` the script. You can use the help option to get details

`python3 collect-logs.py --help`

### Step 2: Execute `collect-logs.py` with required arguments for downloading log files from `CustomerDB`

```bash
$ python3 collect-logs.py --db CustomerDB --customer-profile prod-saas-customers --customer-db-region eu-central-1 --db-uuid _7FF9IjNQCGv7yVS_o5tMg --cluster-uuid tgM7UCGxSV2x8Dua3qFYWA --log-prefix Support-00001 --debug-info 1 --exasolution-log-type 3 --start-time 2021-09-30 --stop-time 2021-09-30

Directory '/srv/bugtrack/SAAS/Support-00001-5172568954/' created.
Starting to download log file 'exacluster_debuginfo_2021_09_30-10_40_32.tar.gz' from s3...
All log files downloaded to '/srv/bugtrack/SAAS/Support-00001-5172568954/' directory.
```

### Note 1: How to omit `--cluster-uuid` and `--exasolution-log-type`

If you don't need to specify the `--cluster-uuid` and `--exasolution-log-type` arguments you can set the value of them as `none`.

### Note 2: Special characters in `--cluster-uuid`

If `--cluster-uuid` starts with dash (`-`) or contains special characters like space, one needs to pass it the following way:

```shell
--cluster-uuid='your value'
```

After downloading log files you can see the path of log files and you can provide this information to the R&D team.

## Additional Notes

This is a temporary solution for getting log files. We will launch a new way to get log files soon and a new article will follow. Detailed information can be found in [Github](https://github.exasol.com/saas/log-management).
