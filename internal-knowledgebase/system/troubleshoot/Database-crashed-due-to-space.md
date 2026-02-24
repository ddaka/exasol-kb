---
tool_name: confd_client
doc_type: troubleshoot
category: system
title: "Database crashed due to space"
summary: "Database crashed and is not starting due to the system running out of disk space. Customers may encounter this issue if the data volume has reached full capacity. You can confirm..."
---
# Database crashed due to space

## Problem

Database crashed and is not starting due to the system running out of disk space. Customers may encounter this issue if the data volume has reached full capacity. You can confirm this by checking the PDD logs and verifying available disk space on the affected node.

## Procedure

1. Check the PDD logs under `/exa/logs/db/<database name>` to verify the cause of the database crash.
2. Run `lsblk` to inspect the disk space usage.
3. Connect to the database COS container using `c4 connect -t 1/cos` and check the size of the data volume with `csinfo -v`.
4. If there is insufficient space, advise the customer to increase the storage capacity.
5. Once the storage has been increased, enlarge the data volume using the follwing command and start the database:

```bash
   confd_client st_device_enlarge node: 11 devname: /dev/vg_exasol/lv_exasol num_sectors: 0
