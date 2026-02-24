---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "INTERNAL - ExaCloud - Installation guide"
summary: "This article guides you through the installation process of a new ExaCloud system."
---
# INTERNAL - ExaCloud - Installation guide

## Overview

This article guides you through the installation process of a new ExaCloud system.

## Prerequisites & Explanation

* The customer must fill a VPN document and send it back to us, please refer to this link for detailed information: [EXASOL VPNs (EN)](https://exasol.atlassian.net/wiki/spaces/IT/pages/19891067/EXASOL+VPNs+EN)
(An alternative option would be public IPs. In this case, no VPN document is required.) For existing customer this is usually not needed.
* A support host has to be set up and configured by ITM (It could be, that an existing support host will be used)
* The hardware must be available and running. The ISO file with the required Exasol image must be mounted on the license server (You can find the servers and their MAC addresses in "i-doit" or in KVM)
* The license file can be found in the Assets for the Account

**Important: Remember to write down all important information about this cluster (e.g logins, passwords, specialties) into Keeper. If a folder for this customer doesn't exist yet, request the creation of one via an SDM Ticket.**

The installation itself basically is the exact same as described in this guide: <https://docs.exasol.com/db/latest/administration/on-premise/installation.htm>

### Prepare installation checklist

* download the checklist from this article from the Download section
* change the name of the checklist to <Cloudno-Installation-LICENSE_NAME>
* upload in Teams Customer 360 -> Customers -> <CUSTOMER_NAME> - if <CUSTOMER_NAME> folder does not exist, create it
* edit the uploaded checklist there

### For the network interfaces

The license server is virtualized. ITM is providing the network configuration with VLAN IDs. The MAC addresses can be found in <https://idoit.core.exasol.com/idoit/>.

In order to have a redundant network we use LACP bond for the data nodes. The rule for the assignment of the VLAN IDs and IPs are:

VLAN tag: {Cloudno.}+1 -> Public
VLAN tag: {Cloudno.}+2 -> Private1

public IP: 172.30.{Cloudno}.XX

Example: We install the cluster on c0025, then it looks like this:
VLAN tag:251 -> Public
VLAN tag:252 -> Private

public IP: 172.30.25.XX

How to configure LACP: [INTERNAL - LACP Setup (IEEE 802.3ad)](internal-lacp-setup-ieee-802-3ad.md)

## Installation process

* Install the license server with disk encryption & install latest Security Patch
* Configure access management - change the default password for EXAoperation user `admin` & set encryption password for the data nodes -> System Passwords
* Configure the cluster network & configure the LACP as described [here](internal-lacp-setup-ieee-802-3ad.md) for the data nodes
* Create monitoring service - Do not forget to add the database later to the log service
* Create the first data node
* Set hugepages for the nodes, Please refer to this article: [INTERNAL - How to calculate hugepages](https://github.com/exasol/Internal-Knowledgebase/blob/main/Environment-Management/internal-how-to-calculate-hugepages.md)
* Create additional nodes with multi copy
* Boot the nodes
* Start EXAStorage & configure the volumes
* Upload the license
* Configure and start the DB - add DB to log service
* Create backup scheduler

There are a few steps that should be done after the installation was completed successfully:

### Steps on the cluster and in the shell

* Change the root password and put it into the Keeper
* Ensure that the network configuration is working. Try to ping every server from every server. Ensure you can connect to the DB.
* Check if there are extra DB parameter for this customer
* Write an initial Level-0 backup
* Create LOM Entry (EXAoperation > Network > SrvMngmt) and attach them to the nodes. Check if the LOM Interface works and you have access to it
* Check the /etc/hosts file on the support host. If there are any entries missing, create a SDM ticket with all information (Cloud number (c00xx), IPs, hostnames)

In EXAplus:

* Create an exa_debug user (<https://docs.exasol.com/db/latest/planning/support.htm#DatabaseAccess>)

 **Inform the customer to change the SYS password immediately when they have access!**

### Steps in Salesforce

* Create a DB and Cluster asset and link them accordingly (Also with LIC asset). Within the Cluster asset, you have to fill in the sizing values of the cluster. If you are unsure, just compare it to other DB or Cluster assets.

### Steps for monitoring

* **Hardware Monitoring (OMSA):**
Follow these instructions: [KB - SYS - Hardware monitoring plugins (Dell servers)](https://exasol.atlassian.net/wiki/spaces/SUPPORT/pages/6755662/KB+-+SYS+-+Hardware+monitoring+plugins+Dell+servers)

* **Install proust**
Follow [How to install and configure proust monitoring](how-to-install-and-configure-proust-monitoring.md)

As soon as all steps from "Install proust" are done, inform L2 System Support team.

## Additional information

* If you need the database to be available through the Internet, set the following database parameter to enforce protocol encryption for EXAplus clients and drivers otherwise traffic between client and database could be unencrypted.

```text
-forceProtocolEncryption=1
```

Since version 8 this parameter is not necessary as by default DB accepts only TLS-encrypted connections (no ChaCha20 or unencrypted): [CHANGELOG: Database accepts only TLS connections](https://exasol.my.site.com/s/article/Changelog-content-16927?language=en_US).

* If there will be a **remote archive volume**, refer to this: <https://docs.exasol.com/administration/on-premise/manage_storage/create_remote_archive_volume.htm>
* If the customer uses **BucketFS**, refer to this: <https://docs.exasol.com/database_concepts/bucketfs/bucketfs_setup.htm>
* If the cluster uses **additional networks**, refer to this: <https://docs.exasol.com/administration/on-premise/installation/prepareenvironment/additional_networks.htm>

## Downloads

[installation checklist](https://exasolcom-my.sharepoint.com/:x:/g/personal/georg_henne_exasol_com/EX5GtuyKzttGl7qpw5O4-bgBcVlvDMi5lKR--B9OcAdl9A?e=rGqslN)
