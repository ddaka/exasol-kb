---
tool_name: cos
doc_type: troubleshoot
category: system
title: "OS Disk Usage Monitor"
summary: "An alert was created as follows:"
---
# OS Disk Usage Monitor

## Problem

An alert was created as follows:

Case Subject: OS Disk Usage Monitor

Case Description:
OS Disk Usage Monitor

## Procedure

Find out why are the disks full and which partitions are affected.
Check filesystem for all nodes with
`df -h`
`psh df -h`
You need to locate which folder is occupying a lot of disk space.
Depending which partition is affected you need to do the following:

- /d00_os partition
1) Are there any old Exasol versions?
Check in EXAoperation -> Software for 'Obsolete EXASuite' packages, if there are any delete them - only on running nodes the packages will be deleted.
2) `/var/log`
Check logs for any large in `/var/log/` & `/var/log/logd/`
The logs in `logd` will be rotated to `/.logbackup/` once a day.
Delete old large logs.
3) EXAoperation metadata
Check for large files in `$COS_DIRECTORY/var/.saved_metadata` and if so delete the oldest.
4) Zope logs
Check size of files in `$COS_DIRECTORY/var/exaoperation/log` and `/.logbackup`, delete if there are any large and older files.
5) old Security packages
Security packages are not automatically deleted. You can find them in `$COS_DIRECTORY/var/exaoperation/clients/packages`.
You can delete all except the currently applied Patch.

**NOTE: if you are using psh to delete files on multiple nodes, you must use the absolute path to the file within the psh command**

- /d02_data
1) coredumps
check coredumps
`/d02_data/coredumps/`
Are there old ones which can be deleted? If so, delete them.
2) database logs
check database logs in
`/d02_data/<DATABASE_NAME>/log/process/.logbackup`
Delete oldest logfiles first.
3) BucketFS
The BucketFS is also saved under `/d02_data/`. You can check the names for the buckets in EXAoperation -> EXABuckets.
Ask the customer to consider deleting files if this is occupying too much space.
Preinstalled "protegrity" files residing in BucketFS are usually small and gain from removal is small.
Sometimes Script Language Container files could be removed, if they aren't used anymore. For details please refer to [Which Script Language Containers are still in use](/Environment-Management/which-script-language-containers-are-still-in-use.md).

**NOTE: if you are using psh to delete files on multiple nodes, you must use the absolute path to the file within the psh command**

- /exa
1) coredumps
check coredumps
`/exa/spool/coredumps/`
Are there old ones which can be deleted? If so, delete them.
2) database logs
check database logs in
`/exa/logs/db/<DATABASE_NAME>/.logbackup`
Delete oldest logfiles first.
3) BucketFS
The BucketFS is also saved under `/exa/data/bucketfs/`.
Ask the customer to consider deleting files if this is occupying too much space.

**NOTE: if you are using psh to delete files on multiple nodes, you must use the absolute path to the file within the psh command**

Afterwards check usage again and find the cause of the large files.

## Additional References

* [Which Script Language Containers are still in use](/Environment-Management/which-script-language-containers-are-still-in-use.md)


