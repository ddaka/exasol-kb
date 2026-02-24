---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "v8 How to install OMSA offline"
summary: "This article is for installing OMSA in version 8 and without direct internet access in the server where it will be installed. More information is shared in the main file (v8 How..."
---
# v8 How to install OMSA offline

## Overview

This article is for installing OMSA in version 8 and without direct internet access in the server where it will be installed.
More information is shared in the main file (v8 How to install OMSA)
The needed files will be downloaded in a server with internet access (referred to as server2) and with the same OS version (a test server in MAOP can be used).
There are 2 servers mentioned on this guide: server1 (the server with no internet access), and server2 (the server with internet access).

## Prerequisites

Check [here](https://linux.dell.com/repo/community/openmanage/) which version applies for your OS and server generation.
You need root and internet access in server2.

## How to install OMSA offline

* **The next steps are in server2** *

1. Create source list.

Create the file '/etc/apt/sources.list.d/linux.dell.com.sources.list'.

    ```bash
    sudo echo 'deb http://linux.dell.com/repo/community/openmanage/{product-version}/{release-name} {release-name} main' | sudo tee -a /etc/apt/sources.list.d/linux.dell.com.sources.list
    ```

For example Ubuntu 22.04.x LTS (jammy) with PowerEdge R7615:

    ```bash
    sudo echo 'deb http://linux.dell.com/repo/community/openmanage/11010/jammy jammy main' | sudo tee -a /etc/apt/sources.list.d/linux.dell.com.sources.list
    ```

2. Add DELL repository key.

To verify OMSA packages, download the repository key from DELL and add it to apt.

    ```bash
    sudo wget https://linux.dell.com/repo/pgp_pubkeys/0x1285491434D8786F.asc
    sudo apt-key add 0x1285491434D8786F.asc
    ```

    NOTE: The key is maintained by DELL. In case DELL updates the key, then this article needs to be updated, ref: <https://linux.dell.com/repo/community/openmanage/>.

3. Update apt repository.

Make apt aware of the new software repository by issuing the following command:

    ```bash
    sudo apt-get update
    ```

4. Download the srvadmin-all packages.
The following will only download the files without installing them:

    ```bash
    sudo apt install --download-only srvadmin-all
    ```

The files will be placed in the following path:

    ```bash
    /var/cache/apt/archives/
    ```

Copy the files from server2 to server1

* **The next steps are in server1** *

1. Copy the files to server1.

Put the files in:

    ```bash
    /var/cache/apt/archives/
    ```

2. Install the packages.

Install all packages at once (Note: There will be errors for some of the packages):

    ```bash
    cd /var/cache/apt/archives/
    sudo dpkg -i *.deb
    ```

3. Reinstall the packages with errors.

Reinstall them in the following order:

    ```bash
    sudo dpkg -i srvadmin-idracadm8_11.0.1.0_amd64.deb
    sudo dpkg -i srvadmin-jre_11.0.1.0_amd64.deb
    sudo dpkg -i srvadmin-tomcat_11.0.1.0_amd64.deb
    sudo dpkg -i srvadmin-webserver_11.0.1.0_amd64.deb
    sudo dpkg -i srvadmin-storelib-sysfs_11.0.1.0_amd64.deb
    sudo dpkg -i srvadmin-storelib_11.0.1.0_amd64.deb
    sudo dpkg -i srvadmin-storage_11.0.1.0_amd64.deb
    sudo dpkg -i srvadmin-storage-snmp_11.0.1.0_amd64.deb
    sudo dpkg -i srvadmin-storage-cli_11.0.1.0_amd64.deb
    sudo dpkg -i srvadmin-storageservices-cli_11.0.1.0_amd64.deb
    sudo dpkg -i srvadmin-storageservices-snmp_11.0.1.0_amd64.deb
    sudo dpkg -i srvadmin-storage_11.0.1.0_amd64.deb
    sudo dpkg -i srvadmin-storageservices_11.0.1.0_amd64.deb
    sudo dpkg -i srvadmin-all_11.0.1.0_amd64.deb
    ```

4. Start OMSA.

Start all the OMSA components with:

    ```bash
    sudo /opt/dell/srvadmin/sbin/srvadmin-services.sh start
    ```

5. Deploy Open Manage check package.

Copy

    ```bash
    check_openmanage-3.7.12.tar.gz
    ```

to the system. It can be found on every support host in

    ```bash
    https://archive.dev.exasol.com/Support/
    scp check_openmanage-3.7.12.tar.gz USER@IP-address:
    ```

6. Extract Open Manage check package.

Extract the package to

    ```bash
    /opt/
    ```

    ```bash
    tar xf /home/exasol/check_openmanage-3.7.12.tar.gz -C /opt/
    ```

7. Test Open Manage.

Test openmanage:

    ```bash
    $ sudo /opt/check_openmanage-3.7.12/check_openmanage
    OK - System: '', SN: 'XXXXXX', 768 GB ram (12 dimms), 13 logical drives, 27 physical drives
    ```
