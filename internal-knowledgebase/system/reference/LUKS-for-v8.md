---
tool_name: internal-knowledgebase
doc_type: reference
category: system
title: "Create LUKS for version 8"
summary: "You want to encrypt the existing disks used by Storage before installing version 8. Also, automatically unlock theses disks during boot. For this article 2 disks for the Storage..."
---
# Create LUKS for version 8

## Overview

You want to encrypt the existing disks used by Storage before installing version 8. Also, automatically unlock theses disks during boot.
For this article 2 disks for the Storage and 1 disk for the OS where used.
While:
/dev/sda -> OS disk
/dev/sdb -> Storage disk
/dev/sdc -> Storage disk

### Perquisites

- Installed OS
- User with sudo rights

## Explanation

1.) Create a keyfile

Create a keyfile in `/etc/`.

```
echo -n “Exasol2023$“ > /etc/exa_keyfile
```

2.) Format the disks

Format disks with `cryptsetup luksFormat`. Make sure you only format the disks you will use for Storage.

```
cryptsetup luksFormat -c aes-xts-plain64 -s 512 -h sha512 /dev/sdb -d /etc/exa_keyfile -y --batch-mode
cryptsetup luksFormat -c aes-xts-plain64 -s 512 -h sha512 /dev/sdc -d /etc/exa_keyfile -y --batch-mode
```

3.) Open the disks

```
cryptsetup luksOpen /dev/sdb exa1 -d /etc/exa_keyfile
cryptsetup luksOpen /dev/sdc exa2 -d /etc/exa_keyfile
```

4.) Automatically unlock disks

In order to automatically unlock the disks the `/etc/crypttab` needs to be edited.
Get the UUID for the disks with `cryptsetup luksUUID`.

```
root@exasol-11:~# cryptsetup luksUUID /dev/sdb
2ff9e30d-611c-4e13-9b37-4a573bfd89ab
```

Now add the UUID to the `/etc/crypttab` with the path to the `keyfile`.

```
# <target name> <source device>         <key file>      <options>
#
exa1 UUID=9c2f31f7-6055-497c-8bda-19dc954669c3 /etc/exa_keyfile luks
exa2 UUID=847da9b1-0025-40d9-8816-fb5753efec5e /etc/exa_keyfile luks
```


