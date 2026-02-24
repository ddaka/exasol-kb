---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "PLAYBOOK - Security during an installation"
summary: "Disk encryption protects information by converting it into unreadable code that cannot be easily deciphered by unauthorized people. It is used to prevent unauthorized access to..."
---
# PLAYBOOK - Security during an installation

### Disk Encrpytion

Disk encryption protects information by converting it into unreadable code that cannot be easily deciphered by unauthorized people. It is used to prevent unauthorized access to data storage.

### Data & License Server

For virtualized servers, it is not necessary to encrypt the disks since it is nearly impossible to get access to the drives.

For other environments, we recommend encrypting all devices & partitions.

### IP addresses

A publicly available IP address should not be set for the cluster. Such an IP address would make your system open for attacks from everyone around the world.

### Users

### OS

During the installation, you need to specify a password for the *maintenance* user.

The *root* user is locked by default after the installation and is only needed for debugging. It can be unlocked with the *maintenance* user.

### EXAoperation

The EXAoperation Web interface comes with the default user *admin*. As soon as it is available, the password should be changed. You can also create additional users with different permissions. Please see <https://docs.exasol.com/administration/on-premise/installation/install_hardware/config_access.htm> for further details.

### Database

Every newly created database has the *sys* user. This user has all the permissions. Therefore, you should change the password immediately and create dedicated users for the daily business.
