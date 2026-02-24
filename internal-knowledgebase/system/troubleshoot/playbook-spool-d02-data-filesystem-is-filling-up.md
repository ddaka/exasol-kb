---
tool_name: cos
doc_type: troubleshoot
category: system
title: "PLAYBOOK: /spool (d02_data) filesystem is filling up"
summary: "Exasol Cluster stores logs under the `/spool` filesystem (`d02_data`) in all of the data nodes. Depending on the cluster itself this filesystems may eventually fill up and run out..."
---
# PLAYBOOK: /spool (d02_data) filesystem is filling up

## Problem

Exasol Cluster stores logs under the `/spool` filesystem (`d02_data`) in all of the data nodes. Depending on the cluster itself this filesystems may eventually fill up and run out of space. For the majority of the Exasol clusters though, the default size (at the time of writing this article: 50 GB) is enough.

From Exasol documentation:

**Spool Disk:**  The spool disk is used for storing database log files and coredumps (ext4 filesystem). Usually this is set to 'd02_data'.

## Diagnosis

When this issue happen, you will see messages like the one shown below at EXAoperation logs page:

```text
n0011.c0001.exacluster.local disk 93.0% full: 'd02_data'
n0012.c0001.exacluster.local disk 85.2% full: 'd02_data'
n0013.c0001.exacluster.local disk 85.7% full: 'd02_data'
```

This example reflects a real case on where the 1st node of the cluster (n0011 in this case), stores more logs than the others. You can see this by:

```text
# psh 'ls -lhtr /d02_data/[DB_NAME]/log/process | wc -l'
0014: 3214
0012: 3203
0013: 3209
0015: 3209
0011: 12012
```

## Explanation

Whenever there is an event that needs to be reported, Exasol logs it on files at the `/spool` filesystem at all of the servers, including the management server. Since the events may happen on one or some of the servers, you might find that the usage of the filesystems differs from one to another. In addition, the first node, usually node n0011 but can be any other, is the one that stores the most of the logs and often is the one that triggers the alert. Per default Node 0 is the only node where SQL texts are logged in full length. All other nodes log only the first 200 characters of a SQL statement.

When you receive such an alert, you may need to take action.

## Recommendation

Please see below what you can do to reduce the disk usage or to prevent this to happen again:

1. Check if the Log rotation is running. Check the `/etc/crontab` file and see if the `cron.daily` task is executed during working hours/normal operation of the cluster to provide proper log rotation
2. Delete old log files like the duplicate ones from previous Exasol versions:

    ```text
    EXASolution-6.1.0:udfplugin_protegrity
    EXASolution-6.1.0:udfplugin_protegrity_664
    EXASolution-6.2.0:udfplugin_protegrity
    EXASolution-6.2.0:udfplugin_protegrity_664
    EXASolution-6.2.1:udfplugin_protegrity
    EXASolution-6.2.1:udfplugin_protegrity_664
    ```

	However, preinstalled "protegrity" files residing in BucketFS are usually small and gain from removal is small.
    Sometimes Script Language Container files could be removed, if they aren't used anymore. For details please refer to [Which Script Language Containers are still in use](which-script-language-containers-are-still-in-use.md).

3. Increase the capacity of `d02_data` on all of the cluster servers. An internal how-to procedure can be found here: [How-to resize d02_data](how-to-resize-d02-data.md). If you do not have access, please get in contact with Exasol Support.

## Additional References

* [Node Properties, Spool Disk](https://docs.exasol.com/db/7.1/administration/on-premise/nodes/configure_node_properties.htm)
* [Storage essentials](https://docs.exasol.com/db/7.1/administration/on-premise/manage_storage/storage_essentials.htm)
* [How-to resize d02_data](how-to-resize-d02-data.md)
* [Which Script Language Containers are still in use](which-script-language-containers-are-still-in-use.md)
