---
tool_name: cos
doc_type: troubleshoot
category: system
title: "Installing IBM Guardium Client on Exasol"
summary: "IBM Guardium prevents leaks from databases, data warehouses and Big Data environments such as Hadoop, ensures the integrity of information and automates compliance controls across..."
---
# Installing IBM Guardium Client on Exasol

## Overview

IBM Guardium prevents leaks from databases, data warehouses and Big Data environments such as Hadoop, ensures the integrity of information and automates compliance controls across heterogeneous environments.

## Prerequisites

* Root access to the Exasol cluster
* IBM Guardium client files downloaded on local machine
* SCP client (scp, WinSCP, etc)

## How to install the IBM Guardium plugin on Exasol

## Upload the IBM Guardium client to all of the nodes

Upload the IBM Guardium installer to the License Server:

```
[user@user_device ~]# scp ./guard-bundle-GIM-11.2.0.10_r109349_v11_2_1-rhel-7-linux-x86_64.gim.sh root@10.114.85.201:/root/
```
Once the file is uploaded, sync across all nodes:

```
[root@n0010 ~]# cos_sync_files /root/guard-bundle-GIM-11.2.0.10_r109349_v11_2_1-rhel-7-linux-x86_64.gim.s
```
When the sync finishes, you should see a message similar to this one:

```
Wed Feb 24 16:59:26 CET 2021: script '/usr/opt/EXASuite-7/EXAClusterOS-7.0.7/sbin/cos_sync_files' finished.
```
## Installing the IBM Guardium client on all of the nodes

Once the sync has finished we will need to create an installation folder for the client on all of the nodes. For that, we will be using parallel shell (psh):

```
psh mkdir -p <folder_for_ibm_guardium>
```
 Once the folders are all created, log in to the data nodes one-by-one via port 20 and install the client:

```
[user@user_device ~]# ssh root@<ip_address_of_node> -p 20
[root@n0010 ~] /root/guard-bundle-GIM-11.2.0.10_r109349_v11_2_1-rhel-7-linux-x86_64.gim.sh <the_parameters_required_for_your_guardium_server>
```

## Notes

For some reason, running the installer inside a for loop or parallel shell remotely doesn't work. Therefore, logging in one-by-one and running the command locally is necessary.

When the installer finishes, you should receive a message similar to the one below:

```
> Installation completed successfully
```

The plugin will stay persistent throughout installs/reboots, unless the nodes are flagged by the "Install Flag"
