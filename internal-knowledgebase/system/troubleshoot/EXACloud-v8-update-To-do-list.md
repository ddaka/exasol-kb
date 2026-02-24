---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "EXACloud v8 update To-do list"
summary: "This is a simple list of task which need to be done for an update of an EXACloud cluster to v8. How to execute these steps is not part of this article!"
---
# EXACloud v8 update To-do list

## Overview

This is a simple list of task which need to be done for an update of an EXACloud cluster to v8.
How to execute these steps is not part of this article!

CURRENT-cluster/DB == original cluster/DB
TEMP-cluster/DB == c0011 cluster/DB

## Explanation

### Preparation

1) TEMP-cluster installed

    Have the TEMP-cluster, which is reachable on c0011, ready with the DB nodes installed.

2) ITM added network config

    The switches are configured so that the TEMP-cluster has access to c0011 and the customer, c00xx, depending on the VLAN configuration for the public network.

3) Create Storage & DB

    Create the volumes in EXAStorage the same as on the CURRENT-cluster.
    Add the database with the same configuration as on the CURRENT-cluster.

4) Add remote Archive Volume

    Create the remote archive Volume on the CURRENT-cluster pointing to the local archive volume on the TEMP cluster.

5) Copy jdbc drivers & bucketFS & certificate

    Recreate the jdbc driver and the bucketFS with the files from the CURRENT-cluster to the TEMP-cluster.
    Also, copy the certificate from the CURRENT-cluster to the TEMP-cluster.

6) Change public IP

    Change the public network in EXAoperation and edit `/etc/rc.local_cos` on the data nodes with the VLAN Tag and the network.
    Make sure you do not have a address conflict here.

7) Backup

    Start a level 0 backup to the TEMP-cluster.

8) Monitoring

    Prepare the monitoring for the TEMP-cluster, including the Dell plugin.

9) Backup scheduler

    Add the backup scheduler to the TEMP-cluster.

### Downtime CURRENT to TEMP

1) Backup

    Start the CURRENT-DB on a different Port. Afterwards, start an INC backup.

2) Restore

    Restore the latest backup into the TEMP-cluster.

3) IP change

    Change the IPs on the CURRENT-cluster.

    After the restore change the IPs on the TEMP-cluster to the old IPs of the CURRENT-cluster. With this the customer does not have to change the connection string.

4) End Downtime

    Inform the customer about the finished restore and ask them to check if they can connect.

5) Backup scheduler

    Activate the backup scheduler.

6) Monitoring

    Activate the monitoring and make sure it is working. (grafana)

### Install v8

1) Sync time

    Alter `/etc/systemd/timesyncd.conf` with the following Exasol NTP servers

    ```text
    NTP=10.42.0.1 10.42.0.53 110.42.0.54
    ```

    Restart the service

    ```shell
    sudo systemctl restart systemd-timesyncd
    ```

2) Network change

    In case the customer is using the same public IPs as we do, 172.30.XX.XX, we can use all interfaces in one LACP bond.
    If so, ask ITM to do this change on the switch.

    Inform ITM to move the iDRACs to the public network.

3) Install OS

    Via the iDRAC install the OS on the servers and encrypt the OS disk.

4) LUKS & LVM

    Follow the knowledge base articles about configuring LUSK & LVMs.
    LUKS comes before LVM.

5) Install v8

    Install version 8 as described in the documentation.

6) Copy jdbc drivers & bucketFS & certificate

    Recreate the jdbc driver and the bucketFS with the files from the TEMP-cluster to the CURRENT-cluster.
    Also, copy the certificate from the TEMP-cluster to the CURRENT-cluster.

    Check with DB support if there anything needed to be done for the JDBC drivers.

7) Configure DB

    Delete the default DB and the default data Volume. Add a new data volume with the correct config.
    Create a new DB with the same name as the previous and the same config.

8) Archive Volume

    Add a local archive volume to the CURRENT-cluster and add this to the TEMP-cluster as a remote volume.

9) Backup scheduler

    Add the backup scheduler to the CURRENT-cluster.

10) Backup

    Write a level 0 backup from the TEMP-cluster to the CURRENT-cluster.

### Downtime TEMP to CURRENT

1) Backup

    Start an INC back from the TEMP-cluster to the CURRENT-cluster.

2) Restore

    Restore the latest backup into the CURRENT-cluster.

3) IP change

    Change the IPs on the TEMP-cluster.

    After the restore change the IPs on the CURRENT-cluster to the old IPs of the CURRENT-cluster. With this the customer does not have to change the connection string.

4) End Downtime

    Inform the customer about the finished restore and ask them to check if they can connect.

5) Monitoring

    Activate the monitoring and make sure it is working. (grafana)

### Clean up

1) Remove DB & Storage

    Delete the TEMP-DB and the volumes in EXAStorage.
    Shutdown EXAStorage and delete the metadata.

2) Wipe

    Set the wipe flag for the DB nodes and restart them.

3) License server

    Inform ITM that the license server of the CURRENT-cluster is no longer needed and can be deleted.

4) Start prepare of TEMP-cluster

    Start the preparation of the TEMP-cluster for the next customer.


