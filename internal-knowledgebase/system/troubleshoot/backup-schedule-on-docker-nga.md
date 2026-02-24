---
tool_name: confd_client
doc_type: troubleshoot
category: system
title: "Backup Schedule on Docker/NGA"
summary: "This article shows 2 ways of creating backup schedules for Docker and NGA installations."
---
# Backup Schedule on Docker/NGA

## Overview

This article shows 2 ways of creating backup schedules for Docker and NGA installations.

## Prerequisites

* Docker, NGA system pre-installed
* SSH access to the cluster/node

## How to set up a backup schedule on Docker/NGA

First, we'll show the traditional method of adding the backup schedule, then we'll show the method by using the ConfD API.

## Step 1. Adding a remote/archive volume

Backups cannot be stored on the data volume. For that you either need an archive volume (local disk) or a remote volume (FTP, SFTP, SMB, AWS S3, Azure Blob, GCS, archive volume on a remote Exasol cluster). We will be configuring an S3 remote volume with an Access/Secret key combination. In order to add a remote archive volume, you need to run the following commands in COS:

```
[root@n11 ~]# exaconf add-remote-volume --name <vol_name> --type <type_of_vol> --owner <exadef_usr_id> --id <not_used_vol_id> --url <storage_url_or_ip> --username <username_or_key_id> --passwd <password_or_secret_key> --options <options>
[root@n11 ~]# exaconf add-remote-volume --name s3_archive --type s3 --owner 500:500 --id 10005 --url https://sea-nga-testingbucket.s3-eu-west-1.amazonaws.com/ --username 123456789ABCDEFG --passwd 123456789ABCDEFGHIJKLMNOPQR --options cleanvolume
```
Once you run the command, you can read your EXAConf file and at the bottom of the file you will see the following section:

```
[RemoteVolume : s3_archive]
    Type = s3
    ID = 10005
    URL = https://sea-nga-testingbucket.s3-eu-west-1.amazonaws.com/
    Owner = 500 : 500
    Username = AKIASXKJGWKU4TNO7P2X
    Passwd = MTIzNDU2Nzg5QUJDREVGR0hJSktMTU5PUFFSCg==
    Options = cleanvolume
```
 The **Passwd** sub-section is base64 encoded. Once you confirm that the configuration is correct, you can commit the config by running the following one-liners:
```
[root@n11 ~]# sed -i '/Checksum =/c\ Checksum = COMMIT' /exa/etc/EXAConf
[root@n11 ~]# exaconf commit

*=== Step 1: synchronizing '/exa/etc/EXAConf' ===*
*--> Successful!*
*=== Step 2: executing 'exalocalconf --commit-local -c /exa/etc/EXAConf 2>&1' ===*
*--> Successful!*
*=== Step 3: creating status file ===*
*--> Successful!*
```
Once that is done, run the following command to view if you can see the contents of the remote volume:
```
[root@n11 ~]# sdfs list <remote_volume_id>
[root@n11 ~]# sdfs list 10005
```
If you get an error, you can add the verbose option to the volume and start troubleshooting the connectivity issues.

## Step 2.1. Adding the backup schedule via the COS CLI

For backups, we usually add a single level-0 backup entry followed by multiple level-1 backup entries. So, usually, we add the level-0 backup to run during a non-work day (Saturday, Sunday) and the level-1 backup to run during the remaining work days (Monday - Friday). This ensures that the resources for the database are not used by the backup process when the database needs it more. For this example, we will be adding a level-0 backup entry for Saturday midnight and 6 level-1 backups for the remaining days.

```
[root@n11 ~]# exaconf add-backup-schedule --backup-name "Full_Backup" --db-name DB1 --volume 10005 --level 0 --expire "9d" --minute "*" --hour 0 --day "*" --month "*" --weekday 6
[root@n11 ~]# exaconf add-backup-schedule --backup-name "Incremental_Backup" --db-name DB1 --volume 10005 --level 1 --expire "3d" --minute "*" --hour 0 --day "*" --month "*" --weekday 0,1,2,3,4,5
[root@n11 ~]# sed -i '/Checksum =/c\ Checksum = COMMIT' /exa/etc/EXAConf
[root@n11 ~]# exaconf commit

*=== Step 1: synchronizing '/exa/etc/EXAConf' ===*
*--> Successful!*
*=== Step 2: executing 'exalocalconf --commit-local -c /exa/etc/EXAConf 2>&1' ===*
*--> Successful!*
*=== Step 3: creating status file ===*
*--> Successful!*
```
Before committing, you can check the EXAConf config file to see the changes under the DB section(s):

```
[DB : DB1]
    Version = 7.0.7
    MemSize = 4 GiB
    Port = 8563
    Owner = 500 : 500
    Nodes = 11,12,13
    NumActiveNodes = 2
    DataVolume = DataVolume1
    AutoStart = True
    [[JDBC]]
        BucketFS = bfsdefault
        Bucket = default
        Dir = drivers/jdbc
    [[Oracle]]
        BucketFS = bfsdefault
        Bucket = default
        Dir = drivers/oracle
    [[Backup : Full_Backup]]
        Enabled = True
        Volume = 10005
        Level = 0
        Minute = *
        Hour = 0
        Day = *
        Month = *
        Weekday = 6
        Expire = 9d
    [[Backup : Incremental_Backup]]
        Enabled = True
        Volume = 10005
        Level = 1
        Minute = *
        Hour = 0
        Day = *
        Month = *
        Weekday = 0,1,2,3,4,5
        Expire = 3d
```
To view the backups you can run the sdfs list command mentioned above. To track the progress of the backup, you can use the following command:

```
[root@n11 DB1]# ls -rt /exa/logs/db/DB1/ | grep PddServer | tail -1 | xargs tail -20f
```
## Step 2.2. Adding the backup schedule via the ConfD API

In this step, we will be achieving the same as above, but using the ConfD API.

First, we need to connect to the cluster via the ConfD API. For that you can refer to [this article](connecting-to-confd-with-python3.md). Once you are connected to the cluster via the ConfD API, run the following commands:

```
>>> conn.job_exec('db_backup_add_schedule', {'params': {'db_name': '*<db_name>*', 'backup_name': '*<backup_name>*', 'backup_volume_id': *<volume_id>*, 'level': *<backup_level>*, 'expire': *<expire_in_days>*, 'minute': *<min>*, 'hour': *<hour>*, 'day': *<day>*, 'month': *<month>*, 'weekday': *<weekday>*, 'enabled': *<enabled_or_disabled>*}})
>>> conn.job_exec('db_backup_add_schedule', {'params': {'db_name': 'DB1', 'backup_name': 'Full_Backup_API', 'backup_volume_id': 10005, 'level': 0, 'expire': '9d', 'minute': '*', 'hour': '0', 'day': '*', 'month': '*', 'weekday': '6', 'enabled': True}})
>>> conn.job_exec('db_backup_add_schedule', {'params': {'db_name': 'DB1', 'backup_name': 'Inc_Backup_API', 'backup_volume_id': 10005, 'level': 1, 'expire': '3d', 'minute': '*', 'hour': '0', 'day': '*', 'month': '*', 'weekday': '0,1,2,3,4,5', 'enabled': True}})
```

| Parameter Name | Type | Example Value | Notes |
| --- | --- | --- | --- |
| db_name | string | DB1 | Name of the Database |
| backup_name | string | Full_Backup | Name of the Backup Config Section |
| backup_volume_id | integer | 10005 | ID of the archive volume / remote volume |
| level | integer | 0 / 1 / 2 ... | Level of Backup |
| expire | string | '0' / '1' / '2' ... ; '*' | Exprie in .. (will be shown in seconds in EXAConf) |
| minute | string | '0' / '1' / '2' ... ; '*' | CRON expression minute |
| hour | string | '0' / '1' / '2' ... ; '*' | CRON expression hour |
| day | string | '0' / '1' / '2' ... ; '*' | CRON expression day |
| month | string | '0' / '1' / '2' ... ; '*' | CRON expression month |
| weekday | string | '0' / '1' / '2' ... ; '*'  | CRON expression weekday |
| enabled | bool | True / False | Enabled or Not? |

Once the command is run, you can check the EXAConf file to verify.

## Option 1. Removing the backup schedule

To remove the backup schedule(s) you can use the following command(s):

### Option 1.1. COS CLI

When inside COS, run the following commands:

```
[root@n11 ~]# exaconf remove-backup-schedule --db-name DB1 --backup-name "Full_Backup"
[root@n11 ~]# exaconf remove-backup-schedule --db-name DB1 --backup-name "Incremental_Backup"
[root@n11 ~]# sed -i '/Checksum =/c\ Checksum = COMMIT' /exa/etc/EXAConf
[root@n11 ~]# exaconf commit

*=== Step 1: synchronizing '/exa/etc/EXAConf' ===*
*--> Successful!*
*=== Step 2: executing 'exalocalconf --commit-local -c /exa/etc/EXAConf 2>&1' ===*
*--> Successful!*
*=== Step 3: creating status file ===*
*--> Successful!*
```
### Option 1.2. ConfD API

First, we need to connect to the cluster via the ConfD API. For that you can refer to [this article](connecting-to-confd-with-python3.md). Once you are connected to the cluster via the ConfD API, run the following commands:

```
>>> conn.job_exec('db_backup_remove_schedule', {'params': {'db_name': 'DB1', 'backup_name': 'Full_Backup_API'}})
```
## Option 2. Modifying the backup schedule

### Option 2.1. Modifying the backup schedule via COS CLI

When inside COS, run the following commands:

```
[root@n11 ~]# exaconf modify-backup-schedule --db-name DB1 --backup-name "Full_Backup" --disabled/enabled --minute 30 --hour 20 --day "*" --month "*" --weekday 0
[root@n11 ~]# sed -i '/Checksum =/c\ Checksum = COMMIT' /exa/etc/EXAConf
[root@n11 ~]# exaconf commit

*=== Step 1: synchronizing '/exa/etc/EXAConf' ===*
*--> Successful!*
*=== Step 2: executing 'exalocalconf --commit-local -c /exa/etc/EXAConf 2>&1' ===*
*--> Successful!*
*=== Step 3: creating status file ===*
*--> Successful!*
```
### Option 2.2. Modifying the backup schedule via ConfD API

First, we need to connect to the cluster via the ConfD API. For that you can refer to [this article](connecting-to-confd-with-python3.md). Once you are connected to the cluster via the ConfD API, run the following commands:

```
>>> conn.job_exec('db_backup_modify_schedule', {'params': {'db_name': 'DB1', 'backup_name': 'Full_Backup_API', 'minute': '30', 'hour': '20', 'weekday': '0'}})
```
## Troubleshooting

## Case 1. Can't connect to the remote archive volume

When you aren't able to perform backups or list the contents of the remote archive volume, you can add the verbose option to it and troubleshoot the issue(s). To do so, edit the EXAConf file:

```
[RemoteVolume : s3_archive]
Type = s3
ID = 10005
URL = https://sea-nga-testingbucket.s3-eu-west-1.amazonaws.com/
Owner = 500 : 500
Username = 123456789ABCDEF
Passwd = 123456789ABCDEF
Options = cleanvolume,verbose
```
Once that is done, commit the config:

```
[root@n11 ~]# sed -i '/Checksum =/c\ Checksum = COMMIT' /exa/etc/EXAConf
[root@n11 ~]# exaconf commi
```
Once all of these changes are commited, a simple sdfs list command:

```
*   Trying 52.218.101.115:443...
* TCP_NODELAY set
* Connected to s3-eu-west-1.amazonaws.com (52.218.101.115) port 443 (#0)
* ALPN, offering http/1.1
* Cipher selection: HIGH:!SSLv2:!SSLv3:!TLSv1:!eNULL:!aNULL:!3DES
* successfully set certificate verify locations:
*   CAfile: /usr/opt/EXASuite-7/EXARuntime-7.0.7/ssl/certs/ca-bundle.crt
  CApath: none
* SSL connection using TLSv1.2 / ECDHE-RSA-AES128-GCM-SHA256
* ALPN, server did not agree to a protocol
* Server certificate:
*  subject: C=US; ST=Washington; L=Seattle; O=Amazon.com, Inc.; CN=*.s3-eu-west-1.amazonaws.com
*  start date: Aug  4 00:00:00 2020 GMT
*  expire date: Aug  9 12:00:00 2021 GMT
*  subjectAltName: host "s3-eu-west-1.amazonaws.com" matched cert's "s3-eu-west-1.amazonaws.com"
*  issuer: C=US; O=DigiCert Inc; OU=www.digicert.com; CN=DigiCert Baltimore CA-2 G2
*  SSL certificate verify ok.
> HEAD / HTTP/1.1
Host: sea-nga-testingbucket.s3-eu-west-1.amazonaws.com
Accept: */*

* Mark bundle as not supporting multiuse
< HTTP/1.1 403 Forbidden
< x-amz-bucket-region: eu-west-1
< x-amz-request-id: 4F317297AA1EB09F
< x-amz-id-2: ri3iLb26buvzBkHyuKEJgs99JUXqHeRk8QPyiwD3m7URgqmWDoPhMXN/3793rFPnEWimD1f62Qg=
< Content-Type: application/xml
< Date: Wed, 10 Mar 2021 13:18:50 GMT
< Server: AmazonS3
<
* Connection #0 to host s3-eu-west-1.amazonaws.com left intact
         SIZE         USAGE         ARCHIVED         MODIFIED           EXPIRE          DELETED   NAME
*   Trying 52.218.65.131:443...
* TCP_NODELAY set
* Connected to sea-nga-testingbucket.s3-eu-west-1.amazonaws.com (52.218.65.131) port 443 (#0)
* ALPN, offering http/1.1
* Cipher selection: HIGH:!SSLv2:!SSLv3:!TLSv1:!eNULL:!aNULL:!3DES
* successfully set certificate verify locations:
*   CAfile: /usr/opt/EXASuite-7/EXARuntime-7.0.7/ssl/certs/ca-bundle.crt
  CApath: none
* SSL connection using TLSv1.2 / ECDHE-RSA-AES128-GCM-SHA256
* ALPN, server did not agree to a protocol
* Server certificate:
*  subject: C=US; ST=Washington; L=Seattle; O=Amazon.com, Inc.; CN=*.s3-eu-west-1.amazonaws.com
*  start date: Aug  4 00:00:00 2020 GMT
*  expire date: Aug  9 12:00:00 2021 GMT
*  subjectAltName: host "sea-nga-testingbucket.s3-eu-west-1.amazonaws.com" matched cert's "*.s3-eu-west-1.amazonaws.com"
*  issuer: C=US; O=DigiCert Inc; OU=www.digicert.com; CN=DigiCert Baltimore CA-2 G2
*  SSL certificate verify ok.
> GET / HTTP/1.1
Host: sea-nga-testingbucket.s3-eu-west-1.amazonaws.com
Accept: */*
Content-Type: application/octet-stream
x-amz-date: 20210310T131850Z
x-amz-content-sha256: e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
Authorization: AWS4-HMAC-SHA256 Credential=123456789ABCDEF/20210310/eu-west-1/s3/aws4_request, SignedHeaders=host;x-amz-date, Signature=649fc0d26245c81e878076dd4bc871ed7f8d8445d0c339121c63febd0eb25942

* Mark bundle as not supporting multiuse
< HTTP/1.1 403 Forbidden
< x-amz-bucket-region: eu-west-1
< x-amz-request-id: 8A50CC17B02E8ED3
< x-amz-id-2: fAU//I7Oi5nRNcIFUy0VABbvwdy+/vnmC/9DbBHbRXC57xYMY0b68u2+ftyXi70jissVHU27jic=
< Content-Type: application/xml
< Transfer-Encoding: chunked
< Date: Wed, 10 Mar 2021 13:18:51 GMT
< Server: AmazonS3
<
* Connection #0 to host sea-nga-testingbucket.s3-eu-west-1.amazonaws.com left intact
```
I simulated a wrong username/password scenario and as you can see from the message above: 403 Forbidden; access to the volume (S3 bucket in this case) is forbidden. Another example:

```
* Mark bundle as not supporting multiuse
< HTTP/1.1 404 Not Found
< x-amz-request-id: B5E34C7AE5774048
< x-amz-id-2: q1ZqyEn5JA8ZDDIRBzcMwfewWJYQCo+zr01fdAAcy6LcAja3XamdJSpoN5v8wE3UiYYfEDu1vtM=
< Content-Type: application/xml
< Date: Wed, 10 Mar 2021 13:27:43 GMT
< Server: AmazonS3
<
* Connection #0 to host s3-eu-west-1.amazonaws.com left intact
```
Here, the URL is wrong and as you see we get a 404 Not Found error. This same method can be applied for adding other options too. Here are some options:

* cleanvolume - deletes remote backups on expiry.
* noverifypeer - doesn’t check server certificate.
* nocompression - writes plain data.
* forcessl - uses STARTTLS in FTP connection.
* webdav - uses WebDAV for http-URL.
* webhdfs - for WebHDFS URLs
* s3 - for servers providing S3 compatible API.
* timeout=seconds - allows higher client/ server response time.
