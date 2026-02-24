---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "DHCPDISCOVER no free leases"
summary: "We recently had a case where the data nodes were failing to boot over PXE due to a network issue with bond0 (private bond). We observed a lot of DHCP events in the..."
---
# DHCPDISCOVER no free leases

## Overview

We recently had a case where the data nodes were failing to boot over PXE due to a network issue with bond0 (private bond). We observed a lot of DHCP events in the `/var/log/all.log` file on the license node:

```bash
<27>1 2023-07-17T16:16:13.822029+01:00 n0010 dhcpd - - -  DHCPDISCOVER from a0:36:9f:78:31:0e via bond0: network 27.1.0.0/16: no free leases
<27>1 2023-07-17T16:16:14.004131+01:00 n0010 dhcpd - - -  DHCPDISCOVER from a0:36:9f:78:35:ba via bond0: network 27.1.0.0/16: no free leases
<27>1 2023-07-17T16:16:17.489911+01:00 n0010 dhcpd - - -  DHCPDISCOVER from a0:36:9f:78:31:0a via bond0: network 27.1.0.0/16: no free leases
<27>1 2023-07-17T16:16:17.999934+01:00 n0010 dhcpd - - -  DHCPDISCOVER from a0:36:9f:78:35:ba via bond0: network 27.1.0.0/16: no free leases
<27>1 2023-07-17T16:16:21.499294+01:00 n0010 dhcpd - - -  DHCPDISCOVER from a0:36:9f:78:31:0a via bond0: network 27.1.0.0/16: no free leases
<27>1 2023-07-17T16:16:22.009312+01:00 n0010 dhcpd - - -  DHCPDISCOVER from a0:36:9f:78:35:ba via bond0: network 27.1.0.0/16: no free leases
<27>1 2023-07-17T16:16:25.508724+01:00 n0010 dhcpd - - -  DHCPDISCOVER from a0:36:9f:78:31:0a via bond0: network 27.1.0.0/16: no free leases
<27>1 2023-07-17T16:16:26.018722+01:00 n0010 dhcpd - - -  DHCPDISCOVER from a0:36:9f:78:35:ba via bond0: network 27.1.0.0/16: no free leases
<27>1 2023-07-17T16:16:30.028054+01:00 n0010 dhcpd - - -  DHCPDISCOVER from a0:36:9f:78:35:ba via bond0: network 27.1.0.0/16: no free leases
<27>1 2023-07-17T16:16:34.037504+01:00 n0010 dhcpd - - -  DHCPDISCOVER from a0:36:9f:78:35:ba via bond0: network 27.1.0.0/16: no free leases
<27>1 2023-07-17T16:16:38.046884+01:00 n0010 dhcpd - - -  DHCPDISCOVER from a0:36:9f:78:35:ba via bond0: network 27.1.0.0/16: no free leases
```

This was caused by an issue with the `private0` interface, even the `bond0` looked healthy:

```bash
[root@n0010]# cat /proc/net/bonding/bond0
Ethernet Channel Bonding Driver: v3.7.1 (April 27, 2011)

Bonding Mode: fault-tolerance (active-backup)
Primary Slave: None
Currently Active Slave: private0
MII Status: up
MII Polling Interval (ms): 1000
Up Delay (ms): 0
Down Delay (ms): 0

Slave Interface: private0
MII Status: up
Speed: 10000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: a0:36:9f:7e:94:18
Slave queue ID: 0

Slave Interface: private0-backup
MII Status: up
Speed: 10000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: a0:36:9f:7e:94:1a
Slave queue ID: 0
```

## Prerequisites

None.

## How to resolve this problem

### Step 1

Connect to the license node via SSH.

### Step 2

Force the `private0-backup` to become the new active interface by running this command:

```bash
ip link set dev bond0 type bond active_slave private0-backup

```

After doing this, the data nodes should boot without other issues via PXE (see the `Currently Active Slave` interface difference)

```bash
[root@n0010 ~]# cat /proc/net/bonding/bond0
Ethernet Channel Bonding Driver: v3.7.1 (April 27, 2011)

Bonding Mode: fault-tolerance (active-backup)
Primary Slave: None
Currently Active Slave: private0-backup
MII Status: up
MII Polling Interval (ms): 1000
Up Delay (ms): 0
Down Delay (ms): 0

Slave Interface: private0
MII Status: down
Speed: 10000 Mbps
Duplex: full
Link Failure Count: 1
Permanent HW addr: a0:36:9f:7e:94:18
Slave queue ID: 0

Slave Interface: private0-backup
MII Status: up
Speed: 10000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: a0:36:9f:7e:94:1a
Slave queue ID: 0
```

## Additional Notes

None.

## Additional References

None.
