---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "v8 How to install OMSA"
summary: "This article is for version 8. In version 7 and before we had the Plugin.Administration.DELL-OpenManage-9.4.0-1.0.9 plugin which could be uploaded via EXAoperation and the..."
---
# v8 How to install OMSA

## Overview

This article is for version 8.
In version 7 and before we had the Plugin.Administration.DELL-OpenManage-9.4.0-1.0.9 plugin which could be uploaded via EXAoperation and the installed.
In version 8 there is no Plugin anymore, therefore we have to install these manually.

## Prerequisites

Check [here](https://linux.dell.com/repo/community/openmanage/) which version applies for your OS and server generation.
You need root and internet access.

## How to install OMSA

1. Create source list

Create the file '/etc/apt/sources.list.d/linux.dell.com.sources.list'.
```
$ sudo echo 'deb http://linux.dell.com/repo/community/openmanage/{product-version}/{release-name} {release-name} main' | sudo tee -a /etc/apt/sources.list.d/linux.dell.com.sources.list
```

For example Ubuntu 22.04.x LTS (jammy) with PowerEdge R7615:
```
$ sudo echo 'deb http://linux.dell.com/repo/community/openmanage/11010/jammy jammy main' | sudo tee -a /etc/apt/sources.list.d/linux.dell.com.sources.list
```

2. Add DELL repository key

To verify OMSA packages, download the repository key from DELL and add it to apt.
```
$ sudo wget https://linux.dell.com/repo/pgp_pubkeys/0x1285491434D8786F.asc
$ sudo apt-key add 0x1285491434D8786F.asc
```

NOTE: The key is mainteined by DELL. In case DELL updates the key, then this article needs to be updated, ref: https://linux.dell.com/repo/community/openmanage/.

3. Update apt repository

Make apt aware of the new software repository by issuing the following command:
```
$ sudo apt-get update
```

4. Install OMSA

Install all OMSA components:
```
$ sudo apt-get install srvadmin-all
```

5. Start OMSA

Start all the OMSA components with:
```
$ sudo /opt/dell/srvadmin/sbin/srvadmin-services.sh start
```

6. Update SNMP community string

In the iDRAC, iDRAC settings -> Services -> SNMP Agent change the SNMP Community Name to your community name string.

7. Configure and start SNMP daemon service

In
```
/lib/systemd/system/snmpd.service
```
change the line
```
ExecStart=/usr/sbin/snmpd -Lsd -Lf /dev/null -u Debian-snmp -g Debian-snmp -I -smux,mteTrigger,mteTriggerConf -f
```
to
```
ExecStart=/usr/sbin/snmpd -Lsd -Lf /dev/null -u Debian-snmp -g Debian-snmp -f
```

Afterwards, reload and restart SNMPD.
```
systemctl daemon-reload
service snmpd restart
```

8. Deploy Open Manage check package

Copy
```
check_openmanage-3.7.12.tar.gz
```
to the system. It can be found on every support host in
```
https://archive.dev.exasol.com/Support/
```
```
$ scp check_openmanage-3.7.12.tar.gz USER@IP-address:
```

9. Extract Open Manage check package

Extract the package to
```
/opt/
```
```
tar xf /home/exasol/check_openmanage-3.7.12.tar.gz -C /opt/
```

10. Test Open Manage

Test openmanage, by replacing &lt;community&gt; to your SNMP community string:
```
$ sudo /opt/check_openmanage-3.7.12/check_openmanage -C <community>
OK - System: '', SN: 'XXXXXX', 768 GB ram (12 dimms), 13 logical drives, 27 physical drives
```
