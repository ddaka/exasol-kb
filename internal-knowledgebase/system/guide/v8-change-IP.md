---
tool_name: confd_client
doc_type: guide
category: system
title: "v8 change IPs"
summary: "You want to change the IP address of a v8 cluster."
---
# v8 change IPs

## Overview

You want to change the IP address of a v8 cluster.

## Prerequisites

- access to OS as root/sudoer
- downtime for the cluster

## How to change IPs

### Step 1

Shutdown the database.

```shell
confd_client db_stop db_name $DATABASE_NAME
```

### Step 2

Stop c4_cloud_command.service on every node.

```shell
systemctl stop c4_cloud_command.service
```

### Step 3

Change the IP in the OS. For example in Ubuntu:

```shell
root@exasol-13:/home/exasol# vim /etc/netplan/00-installer-config.yaml
# This is the network config written by 'subiquity'
network:
  ethernets:
    ens18:
      addresses:
      - 10.70.0.223/24
      gateway4: 10.70.0.2
      nameservers:
        addresses: [8.8.8.8]
        search: []
    ens19:
      dhcp4: true
  version: 2

  root@exasol-13:/home/exasol# netplan apply

```

### Step 4

Change the IP for c4. There are 3 locations in the `/var/lib/ccc/etc/c4.yaml` file where you have to change it. This needs to be done for every node.

```shell
exasol@exasol-13:~$ vim /var/lib/ccc/etc/c4.yaml
245-host:
246-    addrs:
247:        - 10.70.0.221
248:        - 10.70.0.222
249:        - 10.70.0.223
250-    datadisk: /dev/mapper
251-    external_addrs:
252:        - 10.70.0.221
253:        - 10.70.0.222
254:        - 10.70.0.223
--
318-    instance_id: ""
319:    instances_ip_addrs: 10.70.0.221,10.70.0.222,10.70.0.223
320-    legacy: true
```

### Step 5

Change the `net.ip` and `net.ips` in the `.ccc/play/local/{CCC_PLAY_ID}/main/{NODE}/conf/cluster.yaml`. This needs to be done for every node.

```shell
net:
  ip: 10.70.0.221
  ips: 10.70.0.221 10.70.0.222 10.70.0.223
```

### Step 6

Adjust the EXAConf file with the new IP and set the Checksum to `COMMIT` on every node for every node.

```shell
1-[Global]
2-    Revision = 1074
3:    Checksum = COMMIT
--
113-[Node : 11]
114:    PrivateNet = 10.70.0.221/16
115:    PublicNet =
116-    Name = n11
126-[Node : 12]
127:    PrivateNet = 10.70.0.222/16
128:    PublicNet =
129-    Name = n12
```

### Step 7

Start c4_cloud_command.service on every node.

```shell
exasol@exasol-12:~$ systemctl start c4_cloud_command.service
```

### Step 8

Check if you can now connect to the DB with the new IP.


