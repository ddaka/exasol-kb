---
tool_name: confd_client
doc_type: troubleshoot
category: system
title: "How to restore storage metadata"
summary: "Do not perform these actions on a PRODUCTION system that doesn't have a REMOTE backup. The consequences to this procedure can lead to COMPLETE DATA LOSS. Be very CAUTIOS while..."
---
# How to restore storage metadata

## DISCLAIMER (READ CAREFULLY)

Do not perform these actions on a PRODUCTION system that doesn't have a REMOTE backup. The consequences to this procedure can lead to COMPLETE DATA LOSS. Be very CAUTIOS while using these commands.

###########################################################################
### ### THIS PROCEDURE HAS BEEN EXECUTED ON A SINGLE NODE SYSTEM ### ###
###########################################################################

## Problem

While trying to enlarge a device using the confd job `st_device_enlarge` the user provided the wrong parameter for `num_sectors`. Ultimately, the device has been enlarged from 1TB to more than 3TB, instead of 2TB. The device itslef has went OFFLINE after restart of the node because Exasol determined that the device has the wrong size and marked it as OFFLINE.

The following error can be seen in the storage logs: `HDD '/dev/exasol_vg/exasol_lv' has only 536869888 sectors (2.00 TiB) but should have 943718400 (3.52 TiB)!`

## Procedure

To recover from this issue, the storage metadata had to be restored from a previous version.

### Stop the Storage services

```
csctrl -d
```

### Rename current metadata file

```
cd /exa/metadata/storage
mv metadata metadata.old
```

### Copy one of the previous working metadata files - preferably one older than a day OR one before enlarging the device itself

```
cp metadata.csctrl.\[2025-10-09_13\:05\:07\] metadata
```

### Start the Storage services

```
csctrl -s -c /exa/etc/cos_storage.conf
```

### After the start of the Storage services, the device/disk should be ONLINE.

```
csinfo -H
```

### Check the device size (it should have the previous size 1TB)

```
csinfo -D
```

### Start the Database to make sure the DB is consistend

```
confd_client db_start db_name: Exasol
```

### Enlarge the device once again

```
confd_client st_device_enlarge node: 11 devname: /dev/exasol_vg/exasol_lv num_sectors: 0
```

### Check if device size has grown

```
csinfo -D
```

## Additional References

[st_device_enlarge](https://docs.exasol.com/db/latest/confd/jobs/st_device_enlarge.htm)
