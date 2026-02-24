---
tool_name: cos
doc_type: troubleshoot
category: system
title: "Azure Disk Not Mounting During Reboot"
summary: "The customer rebooted the data nodes for maintenance and after the reboot the disk devices are not mounted properly anymore."
---
# Azure Disk Not Mounting During Reboot

## Problem (SF case: 00081165)

The customer rebooted the data nodes for maintenance and after the reboot the disk devices are not mounted properly anymore.

EXAoperation logs show the following errors for the "affected" node:
```
HDD mount failed: \r \rDisk /dev/azure_disk_1 primary partitions failed.
/dev/azure_disk_1 primary partition failed.
```

## Procedure

1. In EXAoperation, go to the "affected" node and apply the `Install` flag to it.

2. Connect to the "affected" node (via `rssh`), and to a "healthy" node (via `ssh`) - Compare the partitions. You'll notice that the "affected" node has less partitions than the "healthy" one.

```bash
cat /proc/partitions
```

3. On the "healthy" node - Print the partition layout of the `/dev/sdb` using the `parted` utility. The disk size should be around 107GB.

```bash
COS$ parted /dev/sdb
(parted) print
```

4. On the "affected" node - Partition the `/dev/sdb` with the same partition sizes that you see on the "healthy" node.

```bash
COS$ parted /dev/sdb
(parted) print
(parted) mkpart primary 8425kB 48.3GB # Ignore at the prompt
(parted) mkpart primary 48.3GB 52.6GB
(parted) mkpart primary 52.6GB 101GB
(parted) quit
```

5. On the "affected" node - If you find a link for `/dev/azure_disk_1` please remove it.

```bash
COS$ ls -l /dev/azure_disk_1
COS$ rm /dev/azure_disk_1
```

6. On the "healthy" node - Print the partition layout of the `/dev/sdc`, `/dev/sdd`, `/dev/sde`, etc, using the `parted` utility.

```bash
COS$ parted /dev/sdd
(parted) print
(parted) quit

COS$ parted /dev/sdd
(parted) print
(parted) quit

COS$ parted /dev/sde
(parted) print
(parted) quit
...
```

7. On the "affected" node - Partition the `/dev/sdc`, `/dev/sdd`, `/dev/sde`, etc with the same partition sizes that you see on the "healthy" node.

```bash
COS$ parted /dev/sdc
(parted) print
(parted) mkpart primary ... ... # Ignore at the prompt
(parted) quit

COS$ parted /dev/sdd
(parted) print
(parted) mkpart primary ... ... # Ignore at the prompt
(parted) quit

COS$ parted /dev/sde
(parted) print
(parted) mkpart primary ... ... # Ignore at the prompt
(parted) quit
...
```

8. On the "affected" node - Run the following script

```bash
bash /etc/azure-create-device-links.sh
```

9. On the "affected" node - List the device links. Run the same command on the "healthy" node. You should see the same output on both nodes.

```bash
ls -l /dev/azure*
```

10. On the "license" node - Remove the following file so the "affected" node can start re-installing.

```bash
cluster1 [root@n0010 ~] rm /exasol/init_started
```

11. The "affected" node should be reinstalled on the next retry.

## Additional References

[Configure Node Properties](https://docs.exasol.com/db/7.1/administration/on-premise/nodes/configure_node_properties.htm)

[Parted Man Page](https://manpages.ubuntu.com/manpages/trusty/man8/parted.8.html)
