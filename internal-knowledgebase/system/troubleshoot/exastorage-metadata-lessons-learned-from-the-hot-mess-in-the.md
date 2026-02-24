---
tool_name: cos
doc_type: troubleshoot
category: system
title: "EXAStorage Metadata (lessons learned from the Hot Mess in the GetResponse POC)"
summary: "First thing first: **DON'T TOUCH THE METADATA! EVER! UNLESS COMPLETELY NECESSARY!**"
---
# EXAStorage Metadata (lessons learned from the Hot Mess in the GetResponse POC)

First thing first: **DON'T TOUCH THE METADATA! EVER! UNLESS COMPLETELY NECESSARY!**

Sounds logical, right? Yes, it does. But some time ago when Apple's Docker cluster (same architecture as 'NGA' Exasol) we were given instructions to remove the storage metadata from the /<path_to_nga>/exa/metadata/storage/ directory by a member of the R&D team, so naturally I thought this was standard procedure, even though it's deleting (with a backup, of course) the metadata files, which doesn't sound like a good idea on any day of the year. After that fixed the Apple cluster, first thought that popped into my mind was "maybe this is standard procedure for the NGA architecture? It's a bug that's fixed this way? Okay, sounds fine" and it worked for me ever since, except for the recent events with GetResponse.

## What happened?

So the Exasol database at GetResponse wasn't booting properly and was giving a completely unrelated error (it is a best guess from the software):

![](images/GR_ERR_1.png)

So what I did was shutdown the exasol service (systemctl stop exasol), clear the old metadata (took a backup of it too, of course) and restart the service (systemctl start exasol). This time it didn't help...

Please refer to [how to identify if the database is corrupt or if the storage metadata has issues](how-to-find-out-if-the-storage-metadata-or-the-database-is.md). This article will give more information on how to nail down the issue.

## Did it work out in the end?

Yes. We were able to restore the cluster the next day by restoring the old metadata. First, we shut down the service, replaced the new metadata with the backed-up one and restarted the storage service:

```bash
[root@n11 ~]# csctrl -d
[root@n11 ~]# cp /exa/metadata/storage/metadata /root/new_metadata_bkp
[root@n11 ~]# mv /root/old_metadata_bkp /exa/metadata/storage/metadata
[root@n11 ~]# csctrl --start --auto-add --auto-restart -n 11,12,13 -c /etc/cos/cos_storage.conf
Successfully started EXAStorage in partition 42.
------ phys. NodeID <-> UUID mapping: ------
11 : 1676CC713038417896A7683B0D68885EC03FD5C0
12 : EB354B9823C841A99C252BA96FE6CD74F28F2C1B
13 : A4B6D6AA02A84040A73F90279FF65EF26EE94147
--------------------------------------------
```
The volume was restored, but the disks were disabled, so I ran the following command to enabled them:

```bash
[root@n11 ~]# cshdd --enable -n <node_id_number> -h <path_to_device>
```
After that I started the database and voila, it worked!

```bash
[root@n11 ~]# dwad_client start-wait DB1 (to start the DB)
[root@n11 ~]# dwad_client (to view the status)
```
We still don't know what really caused the issue, but the logs are at R&D for analysis atm.

**But why did it work before?**

Coincidence. Up until this moment it was all a coincidence! I was lucky that the new volume created after deleting the old metadata didn't overwrite the customers' valuable data. So basically, it works like this (from what I understand):

***1******. The reason why I got lucky:***

So, when I clear the metadata and reboot the exasol service, the volume gets recreated from scratch and I was lucky (up until now) that the new volume was greater (or the same size). So something like this:

![](images/AAA.png)

***2. The reason why the "clear metadata" method failed with GetResponse:***

![](images/BBB.png)

So, since the "newly created" volume was smaller than the actual data size and previous volume size, it messed up the volume and storage of the cluster.

**Conclusion:**

Never touch the metadata files, it might be an easy way to fix some issues with Docker/NGA, but it's **NOT** a solution!

**Nice commands we used during the process:**

```bash
# Show info about all volumes:
[root@n11 ~]# csinfo -v
# Show info about specific volume:
[root@n11 ~]# csinfo -v -i <volume_id>
# Show detailed info about HDDs on nodes:
[root@n11 ~]# csinfo -D
# Show status info about HDDs on nodes:
[root@n11 ~]# csinfo -H
# Show recovery status of specific volume:
[root@n11 ~]# csrec -l -v <volume_id>
# Show recovery status of all volumes:
[root@n11 ~]# csrec -l
# Show recovery status of specific volume with percentage:
[root@n11 ~]# csrec -s -v <volume_id>
# Show the contents of a metadata file:
[root@n11 ~]# csmd -p -f <path_to_metadata_file>
# Move volume from node X to node Y:
[root@n11 ~]# csmove -s <source_node_x> -d <destination_node_y> -m -v <volume_id>
# Show nodes and their roles in the database:
[root@n11 ~]# dwad_client sys-nodes <db_name>
# Show the setup of a database:
[root@n11 ~]# dwad_client print-setup <db_name>
# Switch nodes roles (active/reserve) in the database:
[root@n11 ~]# dwad_client switch-nodes <db_name> <active_node> <reserve_node>
# Show the status of the nodes:
[root@n11 ~]# cosps -N
# Enable disks:
[root@n11 ~]# cshhd --enable -n 11 -h <path_to_device>
# Disable disks:
[root@n11 ~]# cshhd --disable -n 11 -h <path_to_device>
```
**Learn from the mistakes of others. You can't live long enough to make them all yourself.**
