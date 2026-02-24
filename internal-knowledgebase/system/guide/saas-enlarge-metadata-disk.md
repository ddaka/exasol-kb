---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "SaaS - Enlarge metadata disk"
summary: "Increase metadata disk capacity on SaaS main cluster nodes by expanding AWS EBS volumes and applying enlargement on each node."
---
# SaaS - Enlarge metadata disk

## Purpose

Main cluster nodes use metadata disks (typically `sdc` and `sdd`). If metadata usage approaches capacity, expand those disks to avoid operational impact.

## Prerequisites

- Access to SaaS production AWS account.
- Cluster UUID (see [`saas-how-to-find-customer-database-uuid-and-cluster-uuid.md`](saas-how-to-find-customer-database-uuid-and-cluster-uuid.md)).
- Access to connect to the cluster container (see [`saas-how-to-connect-to-the-container.md`](saas-how-to-connect-to-the-container.md)).

## Procedure

1. In AWS console, identify the cluster main nodes.

Note: only the main cluster carries metadata disks.

2. For each affected node, open `Storage` and modify metadata volumes (`sdc`, `sdd`) to the required size.

3. Wait until all volume modifications are completed in AWS.

4. Connect to the cluster container and enlarge devices on all target nodes:

```bash
for i in {11..14}; do
  ssh n$i cshdd --enlarge --node-id $i -h /dev/sdc
  ssh n$i cshdd --enlarge --node-id $i -h /dev/sdd
done
```

5. Validate enlargement output per node/device, for example:

```bash
Successfully enlarged HDD /dev/sdc on node 11.
Successfully enlarged HDD /dev/sdd on node 11.
Successfully enlarged HDD /dev/sdc on node 12.
Successfully enlarged HDD /dev/sdd on node 12.
Successfully enlarged HDD /dev/sdc on node 13.
Successfully enlarged HDD /dev/sdd on node 13.
Successfully enlarged HDD /dev/sdc on node 14.
Successfully enlarged HDD /dev/sdd on node 14.
```

## Notes

- For root/home disk capacity workflows, use [`saas-disk-usage-root-and-home-disks.md`](../troubleshoot/saas-disk-usage-root-and-home-disks.md).
- Run the enlargement command for every affected node and both metadata devices.

## References

- AWS EBS volume modification:
  <https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/requesting-ebs-volume-modifications.html#modify-ebs-volume>
- COS storage context:
  `documents/cos/confd-storage-devices.md`
