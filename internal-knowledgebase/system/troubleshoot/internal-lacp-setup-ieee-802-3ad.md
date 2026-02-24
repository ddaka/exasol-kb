---
tool_name: cos
doc_type: troubleshoot
category: system
title: "INTERNAL - LACP Setup (IEEE 802.3ad)"
summary: "DISCLAIMER: This setup does not guarantee more bandwidth as we cannot influence which interface the bonding driver will use. If you're looking for more bandwidth go for dedicated..."
---
# INTERNAL - LACP Setup (IEEE 802.3ad)

## LAG (IEEE 802.3ad - LACP)

 DISCLAIMER: This setup does not guarantee more bandwidth as we cannot influence which interface the bonding driver will use. If you're looking for more bandwidth go for dedicated network interfaces, LACP will not help you!

Those setups listed below are the most common ones but there are scenarios in which other setups might make more sense.
Used LACP options in this tutorial are:

* **mode=4**// Sets an IEEE 802.3ad dynamic link aggregation policy. Creates aggregation groups that share the same speed and duplex settings. Transmits and receives on all slaves in the active aggregator. Requires a switch that is 802.3ad compliant.
* **miimon=100**// Specifies (in milliseconds) how often MII link monitoring occurs. This is useful if high availability is required because MII is used to verify that the NIC is active.
* **lacp_rate=1**// Specifies that partners should transmit LACPDUs every 1 second.
* **xmit_hash_policy=layer3+4**// Uses upper layer protocol information (when available) to generate the hash. This allows for traffic to a particular network peer to span multiple slaves, although a single connection will not span multiple slaves.

Official RedHat Docu

[RedHat Docu Channel Bonding](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/networking_guide/sec-using_channel_bonding "Follow")
[RedHat Docu Multiple Bonds](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/networking_guide/sec-network_bonding_using_the_command_line_interface#sec-Creating_Multiple_Bonds "Follow")
[RedHat Docu Using Channel Bonding](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/networking_guide/sec-using_channel_bonding "Follow")

### General requirements

* An 802.3ad capable networking switch
* The networking switch must support port mode "hybrid" (tagged and untagged VLANs on the same switch port), Option 2
* 10G network cards of all nodes must support PXE with tagged VLANs, Option 2
* LACP ungroup channel-group feature for PXE boot on tagged VLANs, Option 2

Vendor-specific wordings:

* "port-channel lacp fallback"
* "lacp ungroup member-independent port-channel <1-128>" DELL
* "lacp suspend-individual ; no lacp graceful-convergence"

– MLAG

* Multi-Chassis LAG, please refer to example configs for DELL S4048[^running-config-vlt-sw2][^running-config-vlt-sw1]
* VLT setup guide [Dell_Force10_S4810_VLT_Technical_Guide.pdf](https://github.com/exasol/Internal-Knowledgebase/files/9990791/Dell_Force10_S4810_VLT_Technical_Guide.pdf)
* For MLAGs it is recommended to enable RSTP
* For general information please refer to attached PDF [Dell_Networking_N_Series_Multichassis_LAG_MLAG.pdf](https://github.com/exasol/Internal-Knowledgebase/files/9990789/Dell_Networking_N_Series_Multichassis_LAG_MLAG.pdf)

– Untagged VLAN for PUBLIC0
– Tagged VLAN for PRIVATE0

– MTUs are set via the "untagged" VLAN Public Interface (EXAoperation/Network MTU)
– Additional tagged VLANs are just added via EXAoperation/Network/Additional Networks (Select - No Bonding) and DUMMY MAC addresses

## Option 1: Separate Public and Private LACP bonds

This setup might offer bandwidth and redundancy while PUBLIC and PRIVATE networks are physically separated from each other.

![](images/option1.PNG)Data Nodes are set up via EXAoperation and /etc/cos/boot_options on the Mgmt. Node.
Mmgt Node is set up from commandline /etc/sysconfig/network-scripts/ifcfg* and/or /etc/rc.local_cos.

### Prerequisites

* Minimum supported Exasol version 6.0.0
* 4 or more Network Interfaces
* EXAoperation/Network/Additional Networks
	+ 1x Additional Network "Bonded with Private"
	+ 1x Additional Network "Bonded with Public"
	+ Nx Additional Network "Bonded with Private or Public"
* Network Switch supporting 802.3ad

### Possible combinations/setups (these are just examples)

* PUBLIC0 (1G + 1G) client connections
* PRIVATE0 (10G + 10G) DB + Storage
* PUBLIC0 (10G + 10G) client connections
* PRIVATE0 (10G + 10G) DB + Storage
* PUBLIC0 (10G + 10G) client connections
* PRIVATE0 (10G + 10G) DB
* PRIVATE1 (10G + 10G) Storage

### Setup data nodes

* Configure 802.3ad on the networking switch/switches
* Create Additional Networks in EXAoperation "Bonded with Private" and "Bonded with Public"
* Assign node MAC addresses to Public, Private, "Bonded with Public" and "Bonded with Private"
* Edit /etc/cos/bootoptions on Mgmt. Node
```
ifbonding="mode=4 miimon=100 lacp_rate=1 xmit_hash_policy=layer3+4"
```
* cos_mkbootimg
* coskillall appserverd
* reboot nodes

### Check if bonds are up and running

* cat /proc/net/bonding/bond0 (PRIVATE0)
* cat /proc/net/bonding/bond1 (PUBLIC0)

### Setup mgmt. node

MASTER Interface PRIVATE0 "/etc/sysconfig/network-scripts/ifcfg-bond0"

```
DEVICE=bond0
NAME=bond0
TYPE=Bond
BONDING_MASTER=yes
IPADDR=10.0.1.10
PREFIX=24
ONBOOT=yes
BOOTPROTO=static
BONDING_OPTS="mode=4 miimon=100 lacp_rate=1 xmit_hash_policy=layer3+4"
MTU=9000
NM_CONTROLLED=no
```

### SLAVE Interfaces eth0, eth2 for PRIVATE0 bond "/etc/sysconfig/network-scripts/ifcfg-eth0, ifcfg-eth2"

/etc/sysconfig/network-scripts/ifcfg-eth0

```
DEVICE=eth0
NAME=bond0-slave
TYPE=Ethernet
ONBOOT=no
MASTER=bond0
SLAVE=yes
MTU=9000
NM_CONTROLLED=no
```

/etc/sysconfig/network-scripts/ifcfg-eth2

```
DEVICE=eth2
NAME=bond0-slave
TYPE=Ethernet
ONBOOT=no
MASTER=bond0
SLAVE=yes
MTU=9000
NM_CONTROLLED=no
```

### Start interfaces (use ifdown to deactivate in-use interfaces)

```
~]# ifup ifcfg-eth0
~]# ifup ifcfg-eth2
~]# ifup ifcfg-bond0
```

### MASTER Interface PUBLIC0 "/etc/sysconfig/network-scripts/ifcfg-bond1"

```
DEVICE=bond1
NAME=bond1
TYPE=Bond
BONDING_MASTER=yes
IPADDR=192.168.1.10
GATEWAY=192.168.1.1
PREFIX=24
ONBOOT=yes
BOOTPROTO=static
BONDING_OPTS="mode=4 miimon=100 lacp_rate=1 xmit_hash_policy=layer3+4"
MTU=9000
NM_CONTROLLED=no
```

### SLAVE Interfaces eth1, eth3 for PUBLIC0 bond "/etc/sysconfig/network-scripts/ifcfg-eth1, ifcfg-eth3"

/etc/sysconfig/network-scripts/ifcfg-eth1

```
DEVICE=eth1
NAME=bond1-slave
TYPE=Ethernet
ONBOOT=no
MASTER=bond1
SLAVE=yes
MTU=9000
NM_CONTROLLED=no
```

/etc/sysconfig/network-scripts/ifcfg-eth3

```
DEVICE=eth3
NAME=bond1-slave
TYPE=Ethernet
ONBOOT=no
MASTER=bond1
SLAVE=yes
MTU=9000
NM_CONTROLLED=no
```

### Start interfaces (use ifdown the deactivate in-use interfaces)

```
~]# ifup ifcfg-eth1
~]# ifup ifcfg-eth3
~]# ifup ifcfg-bond1
```

## Option 2: Shared Public and Private LACP bond

This setup might offer bandwidth and redundancy while PUBLIC and PRIVATE networks share their networks. As interfaces are shared a VLAN tag needs to be added for each PRIVATE network.

![](images/option2.PNG)

Data Nodes are set up via EXAoperation and /etc/cos/boot_options on the Mgmt. Node.
Mmgt Node is set up from commandline /etc/sysconfig/network-scripts/ifcfg* and/or /etc/rc.local_cos.

* (Optional) Tagged Public Interface on the data nodes via /etc/rc.local_cos [setup tagged vlan using ip command](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/networking_guide/sec-configure_802_1q_vlan_tagging_using_the_command_line "Follow") ### Requirements:
* Minimum supported Exasol version 6.0.0
* 2 or more Network Interfaces
* 1 Public and 1 Private Interface
* Network Switch supporting 802.3ad and 802.3q
* Possible combinations/setups (these are just examples)
	+ PUBLIC0 + PRIVATE0 (10G + 10G) client connections + DB + Storage

### Setup data nodes

* Configure 802.3ad on the networking switch/switches
* Configure 802.3q VLAN tagging for all LACP port channels
* Assign node MAC addresses to Public and Private
* Edit /etc/cos/bootoptions on Mgmt. Node (intifvlan is the VLAN ID of the private VLAN)

```
intifvlan=1234 ifbonding="mode=4 miimon=100 lacp_rate=1 xmit_hash_policy=layer3+4"
```

* cos_mkbootimg
* coskillall appserverd
* reboot nodes

### Setup mgmt. node

MASTER Interface PUBLIC0/PRIVATE0 "/etc/sysconfig/network-scripts/ifcfg-bond01"

```
DEVICE=bond01
NAME=bond01
TYPE=Bond
BONDING_MASTER=yes
IPADDR=192.168.1.10
GATEWAY=192.168.1.1
PREFIX=24
ONBOOT=yes
BOOTPROTO=static
BONDING_OPTS="mode=4 miimon=100 lacp_rate=1 xmit_hash_policy=layer3+4"
MTU=9000
NM_CONTROLLED=no
```

### SLAVE Interfaces eth0, eth1 for PUBLIC0/PRIVATE0 bond "/etc/sysconfig/network-scripts/ifcfg-eth0, ifcfg-eth1"

/etc/sysconfig/network-scripts/ifcfg-eth0

```
DEVICE=eth0
NAME=bond01-slave
TYPE=Ethernet
ONBOOT=no
MASTER=bond01
SLAVE=yes
MTU=9000
NM_CONTROLLED=no
```

/etc/sysconfig/network-scripts/ifcfg-eth1

```
DEVICE=eth1
NAME=bond01-slave
TYPE=Ethernet
ONBOOT=no
MASTER=bond01
SLAVE=yes
MTU=9000
NM_CONTROLLED=no
```

/etc/sysconfig/network-scripts/ifcfg-bond01.1234

```
DEVICE=bond01.1234
NAME=bond01.1234
ONPARENT=yes
IPADDR=10.0.1.10
PREFIX=24
ONBOOT=yes
VLAN=yes
MTU=9000
NM_CONTROLLED=no
```

### Start interfaces (use ifdown the deactivate in-use interfaces)

```
~]# ifup ifcfg-eth0
~]# ifup ifcfg-eth1
~]# ifup ifcfg-bond01
~]# ifup ifcfg-bond01.1234
```

## Downloads

[Dell_Networking_N_Series_Multichassis_LAG_MLAG.pdf](https://github.com/exasol/Internal-Knowledgebase/files/9990789/Dell_Networking_N_Series_Multichassis_LAG_MLAG.pdf)

[Dell_Force10_S4810_VLT_Technical_Guide.pdf](https://github.com/exasol/Internal-Knowledgebase/files/9990791/Dell_Force10_S4810_VLT_Technical_Guide.pdf)

[VLT-LACP-setup-option1.pdf](https://github.com/exasol/Internal-Knowledgebase/files/9990792/VLT-LACP-setup-option1.pdf)

[frazier_01_0407.pdf](https://github.com/exasol/Internal-Knowledgebase/files/9990793/frazier_01_0407.pdf)
