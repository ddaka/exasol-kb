---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Configure 802.1Q VLAN Tagging Using ip Commands"
summary: "The ifconfig command is not usable in this context because the corresponding ifcfg-files do not exist."
---
# Configure 802.1Q VLAN Tagging Using ip Commands

The ifconfig command is not usable in this context because the corresponding ifcfg-files do not exist.

To create an 802.1Q VLAN interface on Ethernet interface eth0, with name VLAN8 and ID 8, issue a command as root as follows:

```
~]# ip link add link eth0 name eth0.8 type vlan id 8
```

To view the VLAN, issue the following command:

```
~]$ ip -d link show eth0.8
4: eth0.8@eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT
     link/ether 52:54:00:ce:5f:6c brd ff:ff:ff:ff:ff:ff promiscuity 0
     vlan protocol 802.1Q id 8 <REORDER_HDR>
```

To remove the VLAN, issue a command as root as follows:

```
ip link delete eth0.8
```
