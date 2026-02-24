---
tool_name: cos
doc_type: troubleshoot
category: system
title: "Change Storage device names to exad"
summary: "Some hardware systems use two or more raid-controllers in such scenarios it might happen that the Linux kernel changes the ordering of the block devices. If that happens, the..."
---
# Change Storage device names to exad

### Overview

Some hardware systems use two or more raid-controllers in such scenarios it might happen that the Linux kernel changes the ordering of the block devices. If that happens, the nodes won't boot after a successful installation.

### Symptoms

Nodes install once but do not reboot. Error message in EXAoperation: "HDD initialisation failed".

### Prerequisites

SSH and root access to all data nodes. EXAoperation admin access.

### Resolution

Install nodes using their standard block device names e.g. /dev/sda, /dev/sdb and so on. Once their are installed set EXAoperation node flag to "TO INSTALL" and change device names of all storage disks from e.g. /dev/sda to /dev/exad1 and /dev/sdb to /dev/exad2 and so on. Use below commands to create device links for all EXAStorage devices. Use psh to execute on all nodes at the same time. Adopt the command if the nodes have more or less disks.

### Identify disks

```bash
for D in /dev/sd{a,b,c,d,e,f,g,h,i}; do psh "hddident -m ${D}1 -n"; done
```
### Create new devices links using **hddident**

```bash
I=1; for D in /dev/sd{a,b,c,d,e,f,g,h,i}; do psh "hddident -m ${D}1 -N /dev/exad$I"; I=$((I+1)); done
```
### Rename devices in EXAoperation

### Set "TO INSTALL" flag for all nodes (do not reboot while this flag is enabled!)

![](images/Screenshot-2021-02-23-at-09.29.29.png)

![](images/Screenshot-2021-02-23-at-09.30.56.png)

### Change device names

![](images/Screenshot-2021-02-23-at-09.34.45.png)

### Change node flags to "ACTIVE"

Reboot nodes

![](images/Screenshot-2021-02-23-at-09.35.45.png)
