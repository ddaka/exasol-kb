---
tool_name: confd_client
doc_type: troubleshoot
category: system
title: "Micro Focus agent +&nbsp;exastablenet (non-root)"
summary: "-> Stablenet is a monitoring agent which one of our customers use, they wanted to use their own monitoring agent. Stablenet has lots of package dependencies that is why R&D..."
---
# Micro Focus agent +&nbsp;exastablenet (non-root)

### FAQ: What is stablenet and what is exastablenet and how is it executed?

-> Stablenet is a monitoring agent which one of our customers use, they wanted to use their own monitoring agent. Stablenet has lots of package dependencies that is why R&D developed a containerised version of it called exastablenet. It is a docker container extract that has Stablenet installed and it mets all Stablenet package requirements.

-> Stablenet itself triggers the Exasol monitoring scripts which are also included in the TAR

-> The Exasol monitoring scripts can be found here <https://github.com/exasol/nagios-monitoring/tree/master/opt/exasol/monitoring>

-> In order to make the scripts work with Stablenet we need to execute scripts and Stablenet using **nsexec**. nsexec creates a new process namespace and binds specified directories - just like e.g. Docker - in our case the Stablenet environment.

This article describes how-to install and configure the Micro Focus agent and Stablenet. All steps need to be executed per node (mgmt. node included). It is also possible to use this Shell script (recommended).

### Shell script for MicroFocus

<https://github.exasol.com/integrated-professional-services/enterprise-scripts/blob/c09c20ef126e7b69b3c2fcab9960f42c1e0c2511/MicroFocusAgent/microfocus_install.sh>

### "Install" exastablenet

Installation in that context is just unpacking the archive as it contains the Stablenet installation and the Exasol monitoring scripts. For more information please read the FAQ

```bash
tar xf exastablenet.tar.gz -C /opt/
```
### Adapt user, pw and DB name in /opt/exastablenet/test.sh

**test.sh** is a wrapper script to test the monitoring scripts and nsexec which are both used by Stablenet to check for the Exasol services states and logservice events. Adapt user, password and database name.

```bash
#!/bin/bash

ROOT=/opt/exastablenet
#set -xe

cp /etc/hosts /etc/resolv.conf $ROOT/root/etc/

env NSEXEC_ROOT_PATH=$ROOT/root/ \
    NSEXEC_TMP_PATH=$ROOT/tmp/ \
    NSEXEC_MOUNT_EXT_A0_PATH=$ROOT/monitoring:/monitoring \
    NSEXEC_MOUNT_EXT_A1_PATH=/var/log:/exalogs \
nsexec_chroot /usr/opt/mountjail /bin/bash -c 'mkdir -p /buckets/cache && python /monitoring/check_services.py -H 127.0.0.1 -u <EXAOPUSER> -p <EXAOPUSERPW>'
env NSEXEC_ROOT_PATH=$ROOT/root/ \
    NSEXEC_TMP_PATH=$ROOT/tmp/ \
    NSEXEC_MOUNT_EXT_A0_PATH=$ROOT/monitoring:/monitoring \
    NSEXEC_MOUNT_EXT_A1_PATH=/var/log:/exalogs \
nsexec_chroot /usr/opt/mountjail /bin/bash -c 'mkdir -p /buckets/cache && python /monitoring/check_db_diskspace.py -H 127.0.0.1 -u <EXAOPUSER> -p <EXAOPUSERPW> -d <DBNAME>'
env NSEXEC_ROOT_PATH=$ROOT/root/ \
    NSEXEC_TMP_PATH=$ROOT/tmp/ \
    NSEXEC_MOUNT_EXT_A0_PATH=$ROOT/monitoring:/monitoring \
    NSEXEC_MOUNT_EXT_A1_PATH=/var/log:/exalogs \
nsexec_chroot /usr/opt/mountjail /bin/bash -c 'mkdir -p /buckets/cache && python /monitoring/check_nodes.py -H 127.0.0.1 -u <EXAOPUSER> -p <EXAOPUSERPW>'
env NSEXEC_ROOT_PATH=$ROOT/root/ \
    NSEXEC_TMP_PATH=$ROOT/tmp/ \
    NSEXEC_MOUNT_EXT_A0_PATH=$ROOT/monitoring:/monitoring \
    NSEXEC_MOUNT_EXT_A1_PATH=/var/log:/exalogs \
nsexec_chroot /usr/opt/mountjail /bin/bash -c 'mkdir -p /buckets/cache && python /monitoring/check_logservice.py -H 127.0.0.1 -u <EXAOPUSER> -p <EXAOPUSERPW> -i 1'
```

### Run script to test nsexec and Stablenet

The script check four things: 1. Are all services ok? 2. Database disk space ok? 3. Are all node online? 4. Any critical logservice messages? For more information about the scripts please refer to the FAQ section -  this is not part of this article.

```bash
cluster1 [root@n0010 exastablenet]# bash test.sh
OK - all node services are OK
OK - Disk space usage of exadb = 0.0%, Usage in GiB = 0.0GiB, Free space = 20.0GiB|usage_percent=0.0%;80.0;90.0 usage=0.0GiB free=20.0GiB temp=0.0GiB temp_usage_ratio=100.0%;60.0;80.0
OK - 2 nodes online
OK - No new messages found
```

## MicroFocus installation

This is just a manual way of installing - recommended way of doing this is using the script.

### Unpack BI RPMs

These are the MicroFocus specific packages provided by the customer.

```bash
tar xf BI_OA_LIN_12.11.011.tar.gz -C /opt/
```
### Prepare OS (run as root)

Create filesystem directories

```bash
mkdir -p /apps/hpbto/hpoa/ov
mkdir -p /apps/hpbto/hpoa/ovdata
mkdir -p /apps/hpbto/hppa/perf
mkdir -p /apps/hpbto/hppa/perfdata
```

Create softlinks

```bash
ln -s /apps/hpbto/hpoa/ov /opt/OV
ln -s /apps/hpbto/hpoa/ovdata /var/opt/OV
ln -s /apps/hpbto/hppa/perf /opt/perf
ln -s /apps/hpbto/hppa/perfdata /var/opt/perf
```

### Install, default DIR /opt

```bash
/opt/OA_LIN_12.11.011/oainstall.sh -i -a -defer_configure
INFO: Log file for installation is at /var/opt/OV/log/oainstall.log
INFO: HP Operations-agent install options are: -install
INFO: Validating pre-requisites for installation on n0010

Requirements:
--------------
[ PASS ] Is user root
[ PASS ] Is Operating system Linux
[ PASS ] Is architecture X64
[ PASS ] Check if DCE Agent is not installed
[ PASS ] Check if m4 is installed
[ PASS ] 350 MB free disk space on /opt/OV
[ PASS ] 75 MB free disk space on /opt/perf
[ PASS ] 160 MB free disk space on /var/opt/OV
[ PASS ] 55 MB free disk space on /var/opt/perf

Recommendations:
-----------------
[ FAIL ] Check if the IPv6/IPv4 address is configured
IPv6/IPv4 address is not configured, check the oainstall.log at /var/op
t/OV/log location.
[ FAIL ] Check if Motif is installed
Motif toolkit not installed, xglance functionality will be affected.

STATUS: All requirements are OK; at least one recommendation has failed.

INFO: n0010 meets all pre-requisites

Server Check:
-----------------

INFO: No server component installed

===================================================================================
INFO: HP Operations agent installation started on - Mon Feb 15 11:03:50 CET 2021
===================================================================================
INFO: Log file for the installation is at : /var/opt/OV/log/oainstall.log
INFO: Installing the HPOvXpl package...
INFO: Installing the HPOvSecCo package...
INFO: Installing the HPOvBbc package...
INFO: Installing the HPOvSecCC package...
INFO: Installing the HPOvCtrl package...
INFO: Installing the HPOvDepl package...
INFO: Installing the HPOvConf package...
INFO: Installing the HPOvPacc package...
INFO: Installing the HPOvPerlA package...
INFO: Installing the HPOvPerfMI package...
INFO: Installing the HPOvGlanc package...
INFO: Installing the HPOvPerfAgt package...
INFO: Installing the HPOvAgtLc package...
^[[BINFO: Installing the HPOvEaAgt package...
INFO: Installing the HPOvOpsAgt package...
====================================================================================
INFO: HP Operations agent installation is complete - Mon Feb 15 11:04:38 CET 2021
====================================================================================

===================================================================================
INFO: HP Operations agent Hotfix: HFLIN_1211003 installation started on - Mon Feb 15 11:04:38 CET 2021
===================================================================================
INFO: Log file for the installation is at : /var/opt/OV/log/oainstall.log
INFO: Installing the package HPOvPerfMI-HFLIN_1211003...
====================================================================================
INFO: HP Operations agent Hotfix installation is complete - Mon Feb 15 11:04:40 CET 2021
====================================================================================

====================================================================================
INFO: HP Operations agent configuration started on - Mon Feb 15 11:04:40 CET 2021
====================================================================================
INFO: Configuring the HPOvXpl component...
INFO: Configuring the HPOvSecCo component...
INFO: Configuring the HPOvBbc component...
INFO: Configuring the HPOvSecCC component...
INFO: Configuring the HPOvCtrl component...
INFO: Configuring the HPOvDepl component...
INFO: Configuring the HPOvConf component...
INFO: Configuring the HPOvPacc component...
INFO: Configuring the HPOvPerlA component...
INFO: Configuring the HPOvPerfMI component...
INFO: Configuring the HPOvGlanc component...
INFO: Configuring the HPOvPerfAgt component...
INFO: Configuring the HPOvAgtLc component...
INFO: Configuring the HPOvEaAgt component...
INFO: Configuring the HPOvOpsAgt component...
======================================================================================
INFO: HP Operations agent configuration is complete - Mon Feb 15 11:04:48 CET 2021
======================================================================================
INFO: Starting the HP Operations agent product...
INFO: Started the HP Operations agent product successfully
```

### Check service state on SSH port 20

```bash
ssh -p20 localhost
cluster1 [root@n0010 ~]# systemctl status ovpa.service
Display all 378 possibilities? (y or n)
cluster1 [root@n0010 ~]# systemctl status ovpa.service
* ovpa.service - OVPA service for OA
Loaded: loaded (/etc/systemd/system/ovpa.service; enabled; vendor preset: disabled)
Active: active (exited) since Mon 2021-02-15 11:05:01 CET; 1min 58s ago
Docs: man:OVPA(1)
Process: 24718 ExecStart=/etc/systemd/ovpa start (code=exited, status=0/SUCCESS)
Main PID: 24718 (code=exited, status=0/SUCCESS)
CGroup: /system.slice/ovpa.service

Feb 15 11:05:00 n0010 systemd[1]: Starting OVPA service for OA...
Feb 15 11:05:01 n0010 systemd[1]: Started OVPA service for OA.
```

### Reconfigure for non-root user

```bash
useradd hpovusr
groupadd hpovgrp
```

### Create HPOV config

```bash
touch /opt/NPU.txt
set eaagt:MODE=NPU
set eaagt:OPC_RPC_ONLY=TRUE
set bbc.cb:SERVER_PORT=60060
set ctrl.sudo:OV_SUDO_USER=hpovusr
set ctrl.sudo:OV_SUDO_GROUP=hpovgrp
```

### Reconfigure agent to use non-root profile

```bash
/opt/OA_LIN_12.11.011/oainstall.sh -a -configure -agent_profile /opt/NPU.txt
INFO: Log file for configuration is at /var/opt/OV/log/oainstall.log
INFO: Checking if HP Operations-agent is installed
INFO: HP Operations-agent is installed, configuration starting
====================================================================================
INFO: HP Operations agent configuration started on - Mon Feb 15 11:18:30 CET 2021
====================================================================================
INFO: Configuring the HPOvXpl component...
INFO: Configuring the HPOvSecCo component...
INFO: Configuring the HPOvBbc component...
INFO: Configuring the HPOvSecCC component...
INFO: Configuring the HPOvCtrl component...
INFO: Configuring the HPOvDepl component...
INFO: Configuring the HPOvConf component...
INFO: Configuring the HPOvPacc component...
INFO: Configuring the HPOvPerlA component...
INFO: Configuring the HPOvPerfMI component...
INFO: Configuring the HPOvGlanc component...
INFO: Configuring the HPOvPerfAgt component...
INFO: Configuring the HPOvAgtLc component...
INFO: Configuring the HPOvEaAgt component...
INFO: Configuring the HPOvOpsAgt component...
======================================================================================
INFO: HP Operations agent configuration is complete - Mon Feb 15 11:24:24 CET 2021
======================================================================================
INFO: Starting the HP Operations agent product...
INFO: Started the HP Operations agent product successfully
INFO: HP Operations-agent configuration is successful
```

### Configure additional agent config

```bash
vi /opt/OV/bin/OpC/install/tsi_oa_activate.conf

HPOAAgentIP=10.183.157.4 # example IP
HPOAManagerIP=10.183.202.23
HPOAProxyIP=0.0.0.0
LOCALNODENAME=inhclexasol01 # example Hostname
```

### Activate service (this takes up to 20 minutes...take a coffee)

```bash
/opt/OA_LIN_12.11.011/tsi_oa_activate.sh
```

### Verify service is running as non-root

```bash
ssh -p20 localhost
ps faux
hpovusr 20541 0.0 0.0 8784 572 ? Ss 11:25 0:00 /opt/perf/bin/ttd
hpovusr 20818 0.0 0.1 1025140 9944 ? Ssl 11:25 0:00 /opt/OV/bin/ovcd
hpovusr 20826 0.0 0.1 683956 8168 ? Sl 11:25 0:00 \_ /opt/OV/bin/ovbbccb -nodaemon
hpovusr 20849 0.0 0.0 478520 7156 ? Sl 11:25 0:00 \_ /opt/OV/lbin/conf/ovconfd
```

### Verify agent config, certs, services

First command should show all services running (check on ssh -p 20 localhost). Second should return the state of the server. Third command should list three certificates.

```bash
/opt/OV/bin/opcagt –status
/opt/OV/bin/bbcutil –ping https://10.183.202.23
/opt/OV/bin/ovcert -list
```

You're done.

### Finalising stablenet setup

We were able to get it running by changing the owner of the stablenet directory which contains all monitoring scripts.
As root run:

```bash
cd /opt/exastablenet
chown -R hpovusr .
```

### Extend PATH and ENVs (hpovusr needs to know about Exasol env)

ADD to /home/hpovusr/.bashrc

```bash
. /etc/cos.conf
export PATH="$COS_DIRECTORY/bin:$COS_DIRECTORY/sbin:/usr/opt/EXASuite-7/EXARuntime-7.0.5/bin:/usr/opt/EXASuite-7/EXARuntime-7.0.5/sbin:$PATH"
```

### Add SU bit if necessary otherwise hpovusr won't be able to run nsexec_chroot (MGMT node only!)

```bash
which nsexec_chroot
/usr/opt/EXASuite-7/EXAClusterOS-7.0.5/sbin/nsexec_chroot

ls -l $(which nsexec_chroot)
-rwxr-xr-x 1 root root 26712 Dec 15 20:47 /usr/opt/EXASuite-7/EXAClusterOS-7.0.5/sbin/nsexec_chroot

chmod u+s $(which nsexec_chroot)
```

### Get ovagent config (obsolete)

```bash
ovconfget eaagt
```

### Setting up the ovagent (obsolete)

```bash
ovconfchg -ns eaagt -set OPC_NODENAME inhasxxxx
ovconfchg -ns eaagt -set OPC_IP_ADDRESS PULBICIP
/opt/OV/bin/ovconfchg -ns sec.core.auth -set MANAGER_ID d5fa44c0-8bb7-75b8-11c6-879a5c69d676
/opt/OV/bin/ovconfchg -ns eaagt.lic.mgrs -set general_licmgr 10.183.202.23 /opt/OV/bin/ovconfchg -ns sec.core.auth -set MANAGER 10.183.202.23
/opt/OV/bin/ovconfchg -ns sec.cm.client -set CERTIFICATE_SERVER 10.183.202.23
```

### Verify setup (working DIR: /opt/OV/bin/)

```bash
ovcert -certreq
```

### Send cert request to server

```bash
ovcert -certreq
```

### List certs

```bash
ovcert -list
```
### Additional stuff - probably manual way to do it (verify)

```bash
10.183.157.4
vi /opt/OV/bin/OpC/install/tsi_oa_activate.conf
HPOAAgentIP=10.183.157.4
HPOAManagerIP=10.183.202.23
HPOAProxyIP=0.0.0.0
LOCALNODENAME=inhclexasol01
```
