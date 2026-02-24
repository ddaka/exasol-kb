---
tool_name: internal-knowledgebase
doc_type: reference
category: system
title: "Minimal SNMP config for HP"
summary: "Minimal SNMP config for HP"
---
# Minimal SNMP config for HP

## Overview

Minimal SNMP config for HP

## Explanation

**#Load HP lib**

```
dlmod cmaX /usr/lib64/libcmaX64.so
```

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
com2sec notConfigUser default aexooCh%euK#oh4
group notConfigGroup v1 notConfigUser
group notConfigGroup v2c notConfigUser
view systemview included .1.3.6.1.2.1.1
view systemview included .1.3.6.1.2.1.25.1.1
access notConfigGroup "" any noauth exact systemview none none
dontLogTCPWrappersConnects yes
```
