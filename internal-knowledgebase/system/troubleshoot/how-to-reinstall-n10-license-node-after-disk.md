---
tool_name: cos
doc_type: troubleshoot
category: system
title: "How-to re-install N10 (license node) after disk encryption failure"
summary: "Our customer's management node is a VMware virtual machine with disk encryption (LUKS) on the disk."
---
# How-to re-install N10 (license node) after disk encryption failure

## Overview

Our customer's management node is a VMware virtual machine with disk encryption (LUKS) on the disk.

The boot process fails after the LUKS password is inserted with an error `/dev/mapper/luks-.... does not exist.`

![](images/image.jpg)

This issue renders N10 not booting anymore and the only option is re-installation of the management node.

## How to reinstall N10 with database and nodes still up and running
1. Disable the link of both (public and private) network interfaces of the VM from vCenter, isolating the network communication with the data nodes
2. Make a note of the private network config from a data node `cat /etc/cos/private_network.cfg` and configure the same during the installation process. The LN can be found in the `hosts` file on a data node `grep n10 /etc/hosts`
3. It's very important to reinstall N10 from the same Exasuite ISO version as the data nodes (including the OS patch)
4. After installation is completed, connect to the LN via the virtual console and stop the 'cos' service (`systemctl stop cos`)
5. Enable the link of both (public and private) network interfaces from vCenter
6. Copy the `/etc/cos/cored_random` file from any of the data nodes `scp -P20 -p /etc/cos/cored_random root@n10:/etc/cos/cored_random`
7. Copy the SSH key pair from any of the data nodes `scp -P20 -pr ~/.ssh root@n10:~`
8. Start the 'cos' service on LN and allow some time for the node to synchronize (`systemctl start cos`)
9. Connect to port 22 `ssh localhost -p22`
10. Sync the storage configuration file `cos_sync_files $COS_DIRECTORY/etc/cos_storage_real.conf`
11. Monitor the logservice. All the cluster configuration files will be synced automatically. When the sync finishes you should see a message like "*Node X[10] has been added to the EXAStorage cluster.*"
12. Move EXAopeartion from the data node to the license node

## Additional References

[Install Exasol on a Hardware - On Premise | Exasol Documentation](https://docs.exasol.com/administration/on-premise/installation/install_hw.htm)

[EXAoperation - On Premise | Exasol Documentation](https://docs.exasol.com/administration/on-premise/admin_interface/exaoperation.htm)

[Exasol Download Section - Downloads - EXASOL User Portal](https://downloads.exasol.com/)
