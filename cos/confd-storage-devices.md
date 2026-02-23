---
tool_name: confd_client
doc_type: reference
category: Storage Devices
subcommands: st_device_add, st_device_remove, st_device_enable, st_device_disable, st_device_enlarge, st_device_info, st_device_modify, st_device_clear_errors, st_node_list, st_node_restore, st_node_resume, st_node_suspend, st_node_set_bg_recovery, st_node_set_bg_recovery_limit, st_node_force_bg_recovery_limit_calibration, st_node_get_link_speed, st_node_set_space_warn_threshold, st_node_stop_restore, st_get_partition_id
---

# confd_client — Storage Devices and Nodes

## Overview

Commands for managing physical storage devices (HDDs/SSDs), storage node operations, background recovery, and partition management.

All commands run inside the COS namespace (SSH port 20002).

## st_device_add

Add a new disk (device) to an existing node.

    seccount: 13107200, sector_size: 4 KiB}

**Permissions**: Groups: root, exaadm, exastoradm

**Parameters**:

- `devname` (str, required): Device name.
- `disk` (str, required): Disk.
- `node` (int, required): Physical node ID.
- `direct_io` (bool, optional): Direct_io in type bool.
- `ephemeral` (bool, optional): Ephemeral in type bool.
- `read_only` (bool, optional): 'If hdd is read_only (default: false).'
- `seccount` (int, optional): 'Device sector count, 0 if unknown (default: 0).'
- `sector_size` (str, optional): 'Device sector size in string or unicode with unit. Example: 4 KiB, default 4 KiB.'
- `sync_crc` (bool, optional): 'CRC sync (default: true).'
- `use_crc` (bool, optional): 'Use CRC (default: true).'

**Examples**:

```bash
confd_client st_device_add {devname: /exa/data/storage/dev.1, disk: default, node: 11, read_only: false,
```

## st_device_remove

Remove a disk (device) on a given node This operation is only allowed if the
disk is currently not in use.

**Permissions**: Groups: root, exaadm, exadbadm

**Parameters**:

- `devname` (str, required): Device name.
- `node` (int, required): Node ID.

**Examples**:

```bash
confd_client st_device_remove {devname: /exa/data/storage/dev.1, node: 11}
```

## st_device_enable

This job enables/re-enables a disk so that it can be used within any volume.
If the disk was already in use and has been disabled, it will be restored if
requested (and if a valid redundancy segment exists).

**Permissions**: Groups: root, exaadm, exadbadm

**Parameters**:

- `devname` (str, required): Device name.
- `node` (int, required): Node ID.

**Examples**:

```bash
confd_client st_device_enable {devname: /exa/data/storage/dev.1, node: 11}
```

## st_device_disable

This job disables the given disk so that it cannot be used anymore (similar 
to a failure). If the disk is currently in use by a volume, all read        
operations are redirected to a valid redundancy segment, if available. If no
redundancy segment is available, the volume will be locked until the disk is
re-enabled.

**Permissions**: Groups: root, exaadm, exadbadm

**Parameters**:

- `devname` (str, required): Device name.
- `node` (int, required): Node ID.

**Examples**:

```bash
confd_client st_device_disable {devname: /exa/data/storage/dev.1, node: 11}
```

## st_device_enlarge

This job will enlarge each segment of the volume by at least the given      
number of blocks, provided that there is enough free space left. Since a    
volume is always equally distributed over all nodes and devices, the        
resulting size may be bigger than requested (but never smaller).

**Permissions**: Groups: root, exaadm, exadbadm

**Parameters**:

- `devname` (str, required): Device name.
- `node` (int, required): Node ID.
- `num_sectors` (int, required): Number of device sectors, 0 if unknown.

**Examples**:

```bash
confd_client st_device_enlarge {devname: /exa/data/storage/dev.1, node: 11, num_sectors: 0}
```

## st_device_info

Return information about a disk identified by node ID and device name. An
exception is thrown if the disk is not found.

**Permissions**: Users: root | Groups: root, exastoradm, exaadm

**Parameters**:

- `devname` (str, required): Device name.
- `node_id` (int, required): Node ID.

**Examples**:

```bash
confd_client st_device_info {devname: /exa/data/storage/dev.1, node_id: 11}
```

## st_device_modify

This job assigns new parameters to a disk. All parameters must be given. If 
sector size and/or the number of sectors are different from the current     
ones, the disk will be resized and (re)initialized, and all data will be    
lost. If the checksum algorithm is modified, the existing checksums are     
reset in order to avoid errors on READ operations.

**Permissions**: Groups: root, exaadm, exadbadm

**Parameters**:

- `devname` (str, required): Device name.
- `node` (int, required): Node ID.
- `disk` (str, optional): Disk name.
- `num_sectors` (int, optional): Number of device sectors.
- `read_only` (bool, optional): Device read only (boolean).
- `sector_size` (str, optional): 'Device sector size, including unit. Example: 1 KiB.'

**Examples**:

```bash
confd_client st_device_modify {devname: /exa/data/storage/dev.1, node: 11, sector_size: 1 KiB}
```

## st_device_clear_errors

Clear device errors on a specified node and device.

**Permissions**: Groups: root, exaadm, exadbadm

**Parameters**:

- `devname` (str, required): Device name.
- `node` (int, required): Node ID.

**Examples**:

```bash
confd_client st_device_clear_errors {devname: /exa/data/storage/dev.1, node: 11}
```

## st_node_list

Return information about all storage nodes.

**Permissions**: Users: root | Groups: root, exastoradm, exaadm

## st_node_restore

Data on all segments (or master segments only) on the given node will be    
restored from their redundancy segments (if available).

**Permissions**: Groups: root, exaadm, exadbadm

**Parameters**:

- `nid` (int, required): Node ID.
- `vid` (int, required): Volume ID.
- `vname` (str, optional): Volume name.

**Examples**:

```bash
confd_client st_node_restore {nid: 11, vname: DataVolume1}
```

## st_node_resume

When the node is resumed, it returns to the online state and is usable      
again. When all suspended nodes of a volume have been resumed, the volume   
will be unlocked (if no other unlock conditions exist).

**Permissions**: Groups: root, exaadm, exadbadm

**Parameters**:

- `nid` (int, required): Node ID.

**Examples**:

```bash
confd_client st_node_resume {nid: 11}
```

## st_node_suspend

This job will suspend the specified node if possible (see below for         
prerequisites). When a node is suspended it will not be marked as offline,  
but I/O operations are not possible and no volume can be created on that    
node. All volumes that contain the suspended node will be locked to prevent 
any I/O operation. A suspended node is in a special state that can only be  
left when the node is either restarted or manually resumed. This can for    
example be used to restart a node without recovering the data. If a         
suspended node is shut down physically, it will not be replaced with another
node and will not be recovered after it has been restarted. Suspending a    
node may be denied if:

**Permissions**: Groups: root, exaadm, exadbadm

**Parameters**:

- `nid` (int, required): Node ID.

**Examples**:

```bash
confd_client st_node_suspend {nid: 11}
```

## st_node_set_bg_recovery

This enables/disables creation of background recovery operations for the    
given node. Copy-on-demand is not affected by this option.

**Permissions**: Users: root | Groups: root, exastoradm, exaadm

**Parameters**:

- `enabled` (bool, required): Enable or disable background recovery.
- `node` (int, required): Node ID of the desired node.

**Examples**:

```bash
confd_client st_node_set_bg_recovery {enabled: true, node: 11}
```

## st_node_set_bg_recovery_limit

This job sets a limit for the throughput of background restoration on the   
given node in MiB/s. EXAStorage will try to reach the given throughput -    
which may not be possible due to hardware limitation or heavy I/O load by   
the DB - but will never exceed it once it has found the proper parameters.  
If the limit is set to 0, the threshold value will be determined            
automatically by EXAStorage.

**Permissions**: Users: root | Groups: root, exastoradm, exaadm

**Parameters**:

- `limit` (int, required): Throughpout limit in MiB/s (0 for automatic selection).
- `nid` (int, required): Node ID of the desired node (can be "all_nodes").

**Examples**:

```bash
confd_client st_node_set_bg_recovery_limit {nid: 11, threshold: 0}
```

## st_node_force_bg_recovery_limit_calibration

This job will force the calibration algorithm for the throughput limit to   
restart.

**Permissions**: Users: root | Groups: root, exastoradm, exaadm

**Parameters**:

- `nid` (int, required): Node ID of the desired node.

**Examples**:

```bash
confd_client st_node_force_bg_recovery_limit_calibration {nid: 11}
```

## st_node_get_link_speed

This job returns the detected speed of the network link on the given node.  
If more than one link exists, the speed of the slowest link is returned.

**Permissions**: Users: root | Groups: root, exastoradm, exaadm

**Parameters**:

- `nid` (int, required): Node ID.

**Examples**:

```bash
confd_client st_node_get_link_speed {nid: 11}
```

## st_node_set_space_warn_threshold

Set a new space warning threshold.

**Permissions**: Users: root | Groups: root, exastoradm, exaadm

**Parameters**:

- `node` (int, required): Node ID of the desired node.
- `threshold` (int, required): The value for the space usage in percent (0 = disabled - 100) at which a warning should be generated.

**Examples**:

```bash
confd_client st_node_set_space_warn_threshold {node: 11, threshold: 100}
```

## st_node_stop_restore

Existing recovery maps on all segments (or master-segments only) on the     
given node will be deleted, and restoration will be stopped. If an affected 
segment/disk fulfills unlock conditions, the volume can be unlocked.

**Permissions**: Groups: root, exaadm, exadbadm

**Parameters**:

- `nid` (int, required): Node ID.
- `vid` (int, required): Volume ID.
- `vname` (str, optional): Volume name.

**Examples**:

```bash
confd_client st_node_stop_restore {nid: 11, vname: DataVolume1}
```

## st_get_partition_id

Return the EXAStorage service partition ID.

**Permissions**: Users: root | Groups: root, exastoradm, exaadm
