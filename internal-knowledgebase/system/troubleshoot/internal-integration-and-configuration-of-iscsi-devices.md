---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "INTERNAL - Integration and configuration of iSCSI devices"
summary: "Integration and configuration of iSCSI devices"
---
# INTERNAL - Integration and configuration of iSCSI devices

## Overview

Integration and configuration of iSCSI devices

## Explanation

Starting with version 6.0, EXASuite clusters support the use of iSCSI devices (see [INTERNAL: Support for iSCSI iBFT based cluster setups](https://exasol.atlassian.net/browse/EXASOL-1436)).

For this to work, the following requirements must be fulfilled:

1. A dedicated network interface must be used for connecting the iSCSI device. Unlike the private, public, or other network interfaces, the MAC of this device does not need to be configured in EXAoperation - it will be determined automatically.
2. The network interface must set an iSCSI iBFT entry via iPXE. Thus, this dedicated network card must be configured to boot before the private network card.
3. Furthermore, due to performance reasons, we recommend using a network with an MTU of 9000 bytes.

As some environments tend to sporadically lose their iSCSI network connectivity, iSCSI devices will always be used as multipath devices (multiple routes to the same iSCSI device) to better withstand such failures. Furthermore, as the order of iSCSI devices cannot be determined beforehand in case of using multiple iSCSI devices, we introduced a naming convention that uses the iSCSI device WWID (World Wide Identifier) to distinguish between them. As a consequence, each device should be named as follows:

```shell
/dev/mapper/iscsi(WWID of first iSCSI device)
/dev/mapper/iscsi(WWID of second iSCSI device)
```
