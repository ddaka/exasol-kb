---
tool_name: confd_client
doc_type: reference
category: system
title: "BucketFS explained"
summary: "BucketFS is a synchronous file system that is available on all database nodes in an Exasol cluster. Each node in the cluster can connect to the BucketFS service and will see the..."
---
# BucketFS explained

## Overview

BucketFS is a synchronous file system that is available on all database nodes in an Exasol cluster. Each node in the cluster can connect to the BucketFS service and will see the same content as the other nodes.
It was built specifically for User Defined Functions (UDFs), to ensure that their instances on all data nodes would have access to the very same custom static files (see [UDF Scripts](https://docs.exasol.com/db/latest/database_concepts/udf_scripts.htm)).

Eventually BucketFS became the way to upload JDBC drivers and Oracle Instant Client to be used in IMPORT / EXPORT commands (see [Driver Management](https://docs.exasol.com/db/latest/administration/on-premise/manage_drivers.htm)).
Driver uploads for deployments with EXAoperation were performed via EXAoperation. For all other deployment types, in particular all deployments starting from DB version 8, driver uploads are performed via BucketFS.

One can find more information about BucketFS in our public documentation: [BucketFS](https://docs.exasol.com/db/latest/database_concepts/bucketfs/bucketfs.htm).

This article is a collection of low level facts about BucketFS that don't always fit into public documentation, but could be useful in daily Support tasks.

## Bucket passwords in v8

Each bucket has a read and a write password. One would use read password to read the bucket content via HTTP(S), if it's not public.
In turn, write password will be needed to upload files to the bucket via HTTP(S).

If a bucket password isn't provided while creating a bucket via ConfD job [bucket_add](https://docs.exasol.com/db/latest/confd/jobs/bucket_add.htm), system will generate a random password.

Current bucket read and write passwords are shown by ConfD job [bucketfs_info](https://docs.exasol.com/db/latest/confd/jobs/bucketfs_info.htm) in Base64-encoded form (attributes `read_passwd` and `write_passwd`):

```text
root@n11:~# confd_client bucketfs_info bucketfs_name: bfsdefault
_sec_name: 'BucketFS : bfsdefault'
buckets:
  ...
  default:
    _sec_name: 'Bucket : default'
    additional_files:
    - EXAClusterOS:/opt/exasol/slc-6.0.0-c4-5-standard-EXASOL-8.0.0/* EXASolution-8.23.1:/opt/exasol/db-8.33.0/bin/udf/*
    name: default
    public: true
    read_passwd: ...
    write_passwd: ...
...
```

Before use (e.g. for uploading files via Curl) the value needs to be Base64-decoded, like running the following in shell

```shell
echo <Base64-encoded value> | base64 -d ; echo
```

## BucketFS settings for deployments with EXAoperation

It is known that for deployments without EXAoperation information on BucketFS could be found via ConfD job [bucketfs_info](https://docs.exasol.com/db/latest/confd/jobs/bucketfs_info.htm).
This information is stored in EXAConf file, but it's always better to use responsible ConfD jobs than working with EXAConf, if they are available.

Deployments with EXAoperation don't have a central text settings store like EXAConf. Therefore, for BucketFS settings one needs to visit EXAoperation UI.

However, EXAoperation UI won't show bucket passwords. Luckily, BucketFS settings for deployments are stored in a dedicated text file having structure similar to EXAConf.
The file is called `bucketfs.cfg` and resides in a folder like `/usr/opt/EXASuite-7/EXAClusterOS-7.1.30/var/exaoperation/cluster1/config`:

```text
cluster1 [root@n0010 ~]# cat /usr/opt/EXASuite-7/EXAClusterOS-7.1.30/var/exaoperation/cluster1/config/bucketfs.cfg
[bfsdefault]
directory = /d02_data/bfsdefault
next-node = :cos
rsync-binary = /usr/bin/rsync
disk = d02_data
http-port = 2580
https-port = 2581
ssl-certfile = /usr/opt/EXASuite-7/EXAClusterOS-7.1.30/var/exaoperation/inst/etc/server_crt.pem
ssl-keyfile = /usr/opt/EXASuite-7/EXAClusterOS-7.1.30/var/exaoperation/inst/etc/server_key.pem
ssl-ca =
cert-verify = none
sync-key = ...
sync-period = 30000
mode = rsync
additional-files = default EXAClusterOS:/usr/opt/EXASuite-7/EXAClusterOS-7.1.30/var/clients/packages/ScriptLanguages-*

[bfsdefault: default]
bucket-id = default
bucket-name = default
public-bucket = True
read-password = ...
write-password = ...
```

## Partitions and logs

For deployments with EXAoperation BucketFS operations are carried out in one of EXAoperation partitions - `appserverd`. Therefore, BucketFS logs (quite limited) could be found in appserverd logs located in `/var/log/logd/`:

```text
cluster1 [root@n0010 ~]# find /var/log/logd -name Appserver*
/var/log/logd/Appserver.log
/var/log/logd/Appserver.log.1
```

In deployments without EXAoperation each BucketFS Service has it's own COS partition with name `bucketfsd-<BucketFS Service name>`:

```text
root@n11:~# cosps
ROOT NODE SUMMARY:
------------------
1 online nodes
0 offline nodes

ID OWNER GROUP PARENT FLAGS ONLINE NODES COMMAND
...
25   500   500      0  RAEI  1/1 bucketfsd-bfsdefault
...
306   500   500     31  RAEI  1/1 bucketfsd-bucketfs1
...
```

BucketFS logs for deployments without EXAoperation are stored in `/exa/logs/cored/`:

```text
root@n11:~# find /exa/logs/cored/ -name bucket*
/exa/logs/cored/bucketfsd.325.25.0.282.log
/exa/logs/cored/bucketfsd.325.306.0.36369.log
```

Seemingly, each startup of each BucketFS Service gets a dedicated log file.

Restarting a BucketFS Service is done by killing the respective partition:

```text
root@n11:~# coskillall bucketfsd-bfsdefault
Send signal 15 to partition 25.
```

## Unpacking and physical location

By design archive files (.zip, .tar, .tar.gz, or .tgz) are always extracted to enable scripts to access the contained files on the local file system (see [Database Access in BucketFS](https://docs.exasol.com/db/latest/administration/on-premise/bucketfs/database_access.htm)).

It's implemented by having files in two locations.

For deployments without EXAoperation the original files are located in `/exa/data/bucketfs/<BucketFS Service name>/<bucket name>/`:

```text
root@n11:~# ls -l /exa/data/bucketfs/bfsdefault/default/
total 699364
-rwxr-xr-x 1 exadefusr exausers     95560 Feb 27 00:29 EXASolution-8.23.1:nschroot
-rwxr-xr-x 1 exadefusr exausers    624056 Feb 27 00:29 EXASolution-8.23.1:udfplugin_applicationprotector_rest
-rwxr-xr-x 1 exadefusr exausers    624056 Feb 27 00:29 EXASolution-8.23.1:udfplugin_applicationprotector_rest_72
-rwxr-xr-x 1 exadefusr exausers    624056 Feb 27 00:29 EXASolution-8.23.1:udfplugin_applicationprotector_rest_72_ua
-rwxr-xr-x 1 exadefusr exausers    624056 Feb 27 00:29 EXASolution-8.23.1:udfplugin_applicationprotector_rest_ua
-rwxr-xr-x 1 exadefusr exausers    620544 Feb 27 00:29 EXASolution-8.23.1:udfplugin_protegrity_rest
-rwxr-xr-x 1 exadefusr exausers    620544 Feb 27 00:29 EXASolution-8.23.1:udfplugin_protegrity_rest_ua
-rwxr-xr-x 2 exadefusr exausers   1460760 Aug 16  2024 drivers:jdbc:mssql:mssql-jdbc-12.6.3.jre8.jar
-rwxr-xr-x 2 exadefusr exausers       164 Aug 16  2024 drivers:jdbc:mssql:settings.cfg
-rwxr-xr-x 2 exadefusr exausers   4036257 Feb 27  2024 drivers:jdbc:oracle:ojdbc8.jar
-rwxr-xr-x 2 exadefusr exausers       146 Feb 27  2024 drivers:jdbc:oracle:settings.cfg
-rwxr-xr-x 1 exadefusr exausers 118888731 Jun 12 19:29 drivers:oracle:instantclient-basic-linux.x64-23.5.0.24.07.zip
-rwxr-xr-x 2 exadefusr exausers 574686625 Mar 27 18:40 exasol-cloud-storage-extension-2.8.6.jar
-rwxr-xr-x 2 exadefusr exausers         0 Mar 31 16:17 jars:myfile.jar
-rwxr-xr-x 2 exadefusr exausers    506916 Jun 12 19:36 virtual-schema-dist-12.0.0-oracle-3.0.5.jar
-rwxr-xr-x 2 exadefusr exausers    509343 Jul 14 23:42 virtual-schema-dist-13.0.0-oracle-3.0.7.jar
-rwxr-xr-x 2 exadefusr exausers   5837172 Dec 17  2024 vs:virtual-schema-dist-10.5.0-exasol-7.1.1.jar
-rwxr-xr-x 2 exadefusr exausers   5838656 Dec 17  2024 vs:virtual-schema-dist-11.0.2-exasol-7.2.0.jar
-rwxr-xr-x 2 exadefusr exausers    499241 Aug 16  2024 vs:virtual-schema-dist-12.0.0-sqlserver-2.1.3.jar
```

One can see that all files reside in the root of bucket folder and folder structure is mimicked via colon `:` sign in file names.

The same files but with archives unpacked and overall normal folder structure are available in `/exa/data/bucketfs/<BucketFS Service name>/.dest/<bucket name>/` (`.dest` subfolder inside BucketFS Service folder):

```text
root@n11:~# ls -l /exa/data/bucketfs/bfsdefault/.dest/default/
total 562236
drwxr-xr-x 2 exadefusr exausers      4096 Jul 16 16:27 EXASolution-8.23.1
drwxr-xr-x 4 exadefusr exausers      4096 Jul 16 16:27 drivers
-rwxr-xr-x 2 exadefusr exausers 574686625 Mar 27 18:40 exasol-cloud-storage-extension-2.8.6.jar
drwxr-xr-x 2 exadefusr exausers      4096 Mar 31 16:17 jars
-rwxr-xr-x 2 exadefusr exausers    506916 Jun 12 19:36 virtual-schema-dist-12.0.0-oracle-3.0.5.jar
-rwxr-xr-x 2 exadefusr exausers    509343 Jul 14 23:42 virtual-schema-dist-13.0.0-oracle-3.0.7.jar
drwxr-xr-x 2 exadefusr exausers      4096 Dec 17  2024 vs
```

There is some kind of synchronization from original files to files in `.dest`. Therefore, changes in original files are automatically propagated to files in `.dest`. As a result, if you want to remove a file from BucketFS not via HTTP(S), but on OS level
(and have a very good reason for such bypassing of the intended way), then remove the original file from `/exa/data/bucketfs/<BucketFS Service name>/<bucket name>/`. That should be done simultaneously on all nodes (say, via `psh`), otherwise file could be re-synchronized back from other nodes.

Deployments with EXAoperation follow the same logic, keeping files in `/d02_data/<BucketFS Service name>/<bucket name>/` and `/d02_data/<BucketFS Service name>/.dest/<bucket name>/` respectively:

```text
cluster1 [root@n0010 ~]# ls -l /d02_data/bfsdefault/default/
total 933660
-rwxr-xr-x 1 exasolution exasolution 941792229 Oct 28  2024 EXAClusterOS:ScriptLanguages-standard-EXASOL-all-slc-v8.1.0-CJDGGFPK.tar.gz
-rwxr-xr-x 2 exasolution exasolution   4036257 May 13 16:00 drivers:jdbc:jdbc1:ojdbc8.jar
-rwxr-xr-x 2 exasolution exasolution   4036257 May 13 16:00 drivers:jdbc:oracle:ojdbc8.jar
-rwxr-xr-x 2 exasolution exasolution    506916 May 13 15:58 virtual-schema-dist-12.0.0-oracle-3.0.5.jar
-rwxr-xr-x 2 exasolution exasolution   5681626 Jun 16 07:31 virtual-schema-dist-9.0.4-oracle-2.2.0.jar

cluster1 [root@n0010 ~]# ls -l /d02_data/bfsdefault/.dest/default/
total 6056
drwxr-xr-x 3 exasolution exasolution    4096 Apr 16 17:21 EXAClusterOS
drwxr-xr-x 3 exasolution exasolution    4096 Jun 16 07:35 drivers
-rwxr-xr-x 2 exasolution exasolution  506916 May 13 15:58 virtual-schema-dist-12.0.0-oracle-3.0.5.jar
-rwxr-xr-x 2 exasolution exasolution 5681626 Jun 16 07:31 virtual-schema-dist-9.0.4-oracle-2.2.0.jar
```

## S3-based BucketFS Service

According to v8 documentation for ConfD job [bucketfs_add](https://docs.exasol.com/db/latest/confd/jobs/bucketfs_add.htm) one can pass the following parameter:

| Parameter name | Data type | Description |
| --- | --- | --- |
| mode | string | One of the following service modes: BucketFS, rsync or SDFS. |

It seems that `rsync` was the only available mode prior to v8.

"SDFS" mode finds use for a particular use case in SaaS. Each SaaS DB has BucketFS Service `uploads`:

```text
root@n10:~# confd_client bucketfs_info bucketfs_name: uploads
_sec_name: 'BucketFS : uploads'
buckets:
  default:
    _sec_name: 'Bucket : default'
    name: default
    public: true
    read_passwd: ...
    write_passwd: ...
bucketvolume: uploads
http_port: 0
https_port: 0
mode: sdfs
name: uploads
owner:
- 500
- 500
sync_key: ...
sync_period: '30000'
```

Important pieces for us are

```text
...
bucketvolume: uploads
...
mode: sdfs
...
```

There is a corresponding remote volume called `uploads`:

```text
root@n10:~# confd_client remote_volume_info remote_volume_name: uploads
name: uploads
options: nocompression,s3-download-part-length=100MiB,
owner:
- 500
- 500
password: ...
type: s3
url: https://<Stack's S3 bucket>.s3.eu-central-1.amazonaws.com
username: ...
vid: '10004'
```

pointing to S3 bucket where the stack's data resides as well.

[Manage UDF Files](https://docs.exasol.com/saas/administration/manage_files.htm) SaaS feature initially uploads files to this bucket, and then those files are copied to the data nodes.

## Additional References

* [BucketFS](https://docs.exasol.com/db/latest/database_concepts/bucketfs/bucketfs.htm)
* [UDF Scripts](https://docs.exasol.com/db/latest/database_concepts/udf_scripts.htm)
* [Driver Management](https://docs.exasol.com/db/latest/administration/on-premise/manage_drivers.htm)
* [BucketFS Jobs](https://docs.exasol.com/db/latest/confd/overview_bucketfs_jobs.htm)
* [Database Access in BucketFS](https://docs.exasol.com/db/latest/administration/on-premise/bucketfs/database_access.htm)
* [Manage UDF Files](https://docs.exasol.com/saas/administration/manage_files.htm)
