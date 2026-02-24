---
tool_name: internal-knowledgebase
doc_type: reference
category: system
title: "Minimal SNMP config for DELL"
summary: "Minimal SNMP config for DELL"
---
# Minimal SNMP config for DELL

## Overview

Minimal SNMP config for DELL

## Explanation

**#POLL INTERFACE: Grants read-only access to the specified node on the public IP address using "aexooCh%euK#oh4" as community string, test it with snmpwalk -v1 aexooCh%euK#oh4 &lt;PUBLIC NETWORK NODE IP ADDRESS&gt;**

```
rocommunity aexooCh%euK#oh4 <PUBLIC NETWORK NODE IP ADDRESS>
```

**#PUSH INTERFACE: Is used to define one or more notification receiver/s e.g. monitoring systems like Nagios**

```
trapcommunity aexooCh%euK#oh4
trapsink <PUBLIC NETWORK NODE IP ADDRESS of the notification receiver>
```

**#Basic settings form the default snmpd.conf, use man snmpd.conf for more information**

```
com2sec notConfigUser   localhost       aexooCh%euK#oh4
group   notConfigGroup v1           notConfigUser
group   notConfigGroup v2c           notConfigUser
view    all            included      .1
view    systemview    included   .1.3.6.1.2.1.1
view    systemview    included   .1.3.6.1.2.1.25.1.1
access  notConfigGroup ""      any       noauth    exact  all    none   none
dontLogTCPWrappersConnects yes
```

**#DELL snmp-agents require a smuxpeer**

```
smuxpeer .1.3.6.1.4.1.674.10892.1
```
