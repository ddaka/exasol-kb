---
tool_name: confd_client
doc_type: reference
category: Volume Management
subcommands: st_volume_create, st_volume_delete, st_volume_info, st_volume_list, st_volume_enlarge, st_volume_change_owner, st_volume_change_permissions, st_volume_add_label, st_volume_remove_label, st_volume_append_node, st_volume_move_node, st_volume_lock, st_volume_unlock, st_volume_clear_data, st_volume_decrease_redundancy, st_volume_increase_redundancy, st_volume_set_forced_red_level, st_volume_set_io_status, st_volume_set_ports, st_volume_set_priority, st_volume_set_shared, remote_volume_add, remote_volume_remove, remote_volume_list, remote_volume_list_details, remote_volume_info, remote_volume_state, object_volume_add, object_volume_info, object_volume_list, object_volume_remove
---

# confd_client — Volume Management

## Overview

Commands for managing local SDFS volumes (data and archive), remote archive volumes (S3, FTP, WebDAV, Azure), and object volumes.

All commands run inside the COS namespace (SSH port 20002).

## st_volume_create

This job creates a new local volume and returns the volume ID. The type of
volume (data or archive) depends on the 'type' parameter value.

**Permissions**: Groups: root, exaadm, exadbadm

**Parameters**:

- `disk` (str, required): Name of the disk to be used for the volume.
- `name` (str, required): Name of the new volume.
- `nodes` (list, required): List of node IDs (integers).
- `redundancy` (int, required): Redundancy level.
- `size` (str, required): 'Volume size with unit in string format. Example: 1 GiB.'
- `type` (str, required): The volume type ('data' or 'archive').
- `block_size` (str|int, optional): 'Block size of the new volume with unit. Example: 64 KiB.'
- `ftp_port` (int, optional): FTP port, on which the archive volume should be accessible.
- `ftps_port` (int, optional): FTPS port, on which the archive volume should be accessible.
- `http_port` (int, optional): HTTP port, on which the archive volume should be accessible.
- `https_port` (int, optional): HTTPS port, on which the archive volume should be accessible.
- `num_master_nodes` (int, optional): Number of master nodes in the new volume.
- `owner` (list|tuple, optional): Owner of the new volume given as (user id, group id). Defaults to (uid of exadefusr, gid of exausers).
- `partition_size` (str|int, optional): The size of the partitions, applies only to archive volumes, default is 4 GiB.
- `permissions` (str, optional): Permissions of the new volume.
- `priority` (str, optional): Volume priority.
- `sftp_port` (int, optional): SFTP port, on which the archive volume should be accessible.
- `shared` (bool, optional): Boolean value which indicating whether the volume is shared.
- `stripe_size` (str|int, optional): 'Stripe size of the new volume. Example: 64 KiB.'

**Examples**:

```bash
confd_client st_volume_create {disk: default, name: new_vol0, nodes: [11], redundancy: 1, size: 1 GiB, type: data}
```

## st_volume_delete

Only the owner of a volume (and root) is allowed to delete it. The volume   
cannot be deleted if it is in use, or if it is a snapshot volume.

**Permissions**: Users: root | Groups: root, exastoradm, exaadm

**Parameters**:

- `vid` (int, required): Id of an existing volume.
- `vname` (str, optional): Volume name.

**Examples**:

```bash
confd_client st_volume_delete {vname: DataVolume1}
```

## st_volume_info

Return information about a volume given the ID or volume name.

**Permissions**: Users: root | Groups: root, exastoradm, exaadm

**Parameters**:

- `vid` (int, required): ID of an existing volume. 'all_volumes' will return information about all existing volumes.
- `vname` (str, optional): Volume name.

**Examples**:

```bash
confd_client st_volume_info {vname: DataVolume1}
```

## st_volume_list

Return a list of all volumes.

**Permissions**: Users: root | Groups: root, exastoradm, exaadm

## st_volume_enlarge

This job will enlarge each segment of the volume by at least the given      
number of blocks, provided that there is enough free space left.

**Permissions**: Users: root | Groups: root, exastoradm, exaadm

**Parameters**:

- `blocks_per_node` (int, required): Number of blocks by which each segment of a volume should be enlarged.
- `vid` (int, required): ID of an existing volume.
- `vname` (str, optional): Name of an existing volume.

**Examples**:

```bash
confd_client st_volume_enlarge {blocks_per_node: 4, vname: DataVolume1}
```

## st_volume_change_owner

Only the owner of a volume (and root) is allowed to change the owner and/or 
group. Partitions that have already opened the specified volume can continue
to use it even if the new permissions would not allow it, but subsequent    
calls to open_volume() will fail.

**Permissions**: Users: root | Groups: root, exastoradm, exaadm

**Parameters**:

- `vid` (int, required): ID of an existing volume.
- `vol_grp` (int, required): ID of an existing group.
- `vol_user` (int, required): ID of an existing user.
- `vname` (str, optional): Name of an existing volume.

**Examples**:

```bash
confd_client st_volume_change_owner {vname: DataVolume1, vol_grp: 1000, vol_user: 1000}
```

## st_volume_change_permissions

Only the owner of a volume (and root) is allowed to change its permissions. 
Partitions that have already opened the specified volume can continue to use
it even if the new permissions would not allow it, but subsequent calls to  
open_volume() will fail.

**Permissions**: Users: root | Groups: root, exastoradm, exaadm

**Parameters**:

- `mode` (str, required): The new permissions.
- `vid` (int, required): ID of an existing volume.
- `vname` (str, optional): Name of an existing volume.

**Examples**:

```bash
confd_client st_volume_change_permissions {mode: rwxrwxrwx, vname: DataVolume1}
```

## st_volume_add_label

A label is an arbitrary string that can be used to identify or classify a   
volume. It has no impact on the operation of the volume. The number of      
labels for a volume is not limited. Labels are by default stored in the same
order as they have been added. Setting the 'front' flag will add the label  
in front of any other labels.

**Permissions**: Users: root | Groups: root, exastoradm, exaadm

**Parameters**:

- `label` (str, required): The label. Can be any string.
- `vid` (int, required): ID of an existing volume.
- `vname` (str, optional): Name of an existing volume.

**Examples**:

```bash
confd_client st_volume_add_label {label: FunVolume, vname: DataVolume1}
```

## st_volume_remove_label

Remove a label from a specified volume.

**Permissions**: Users: root | Groups: root, exastoradm, exaadm

**Parameters**:

- `label` (str, required): The label to remove.
- `vid` (int, required): ID of an existing volume.
- `vname` (str, optional): Name of an existing volume.

**Examples**:

```bash
confd_client st_volume_remove_label {label: FunVolume, vname: DataVolume1}
```

## st_volume_append_node

With this resizing method, the blocks on the new nodes are appended to the  
current address space of the volume. The new space is instantly available   
without performance decrease, but the size and redundancy of the new nodes  
must be identical to the existing ones. Therefore, the number of new nodes  
must be at least as big as the redundancy of the volume.

**Permissions**: Users: root | Groups: root, exastoradm, exaadm

**Parameters**:

- `node_ids` (list, required): List of node IDs.
- `node_num` (int, required): Number of nodes.
- `vid` (int, required): Id of an existing volume.
- `vname` (str, optional): Volume name.

**Examples**:

```bash
confd_client st_volume_append_node {node_ids: [12], node_num: 1, vid: 0}
```

## st_volume_move_node

For each node that you want to move, you must specify a destination node.   
The 'src_nodes' and 'dest_nodes' must be of equal size. The nodes to be     
moved can be online or offline, the destination nodes have to be online.

**Permissions**: Users: root | Groups: root, exastoradm, exaadm

**Parameters**:

- `dst_nodes` (list, required): List of the destination node physical IDs.
- `src_nodes` (list, required): List of the source node physical IDs.
- `vid` (int, required): Volume ID.
- `vname` (str, optional): Volume name.

**Examples**:

```bash
confd_client st_volume_move_node {dst_nodes: [12], src_nodes: [11], vname: DataVolume1}
```

## st_volume_lock

This job locks the specified volume. Manually locking a volume does not add 
an unlock condition, so the volume will never be automatically unlocked. If 
a volume is locked, it will also be closed for all current users.

**Permissions**: Users: root | Groups: root, exastoradm, exaadm

**Parameters**:

- `vid` (int, required): Volume ID.
- `vname` (str, optional): Name of an existing volume.

**Examples**:

```bash
confd_client st_volume_lock {vname: DataVolume1}
```

## st_volume_unlock

This job unlocks the specified volume. Manually unlocking a volume clears   
all existing unlock conditions. The cause of the unlock conditions will     
however remain unchanged. For example, if the volume has been locked because
of a Device failure, the Device will remain offline.

**Permissions**: Users: root | Groups: root, exastoradm, exaadm

**Parameters**:

- `vid` (int, required): Volume ID.
- `vname` (str, optional): Name of an existing volume.

**Examples**:

```bash
confd_client st_volume_unlock {vname: DataVolume1}
```

## st_volume_clear_data

This job overwrites data with zeroes on all segments that reside on the     
specified nodes.

**Permissions**: Users: root | Groups: root, exastoradm, exaadm

**Parameters**:

- `node_ids` (list, required): Node ID of the nodes whose segments should be overwritten. Empty list means all nodes.
- `num_bytes` (int, required): Number of bytes per segment to overwrite. Zero means all bytes.
- `vid` (int, required): Volume ID.
- `vname` (str, optional): Name of an existing volume.

**Examples**:

```bash
confd_client st_volume_clear_data {node_ids: [0], num_bytes: 0, vname: DataVolume1}
```

## st_volume_decrease_redundancy

This job will decrease the redundancy of a given volume by a given level by 
deleting existing redundancy segments. The action may be denied if some of  
the redundancy segments are used for recovering another segment and no other
suitable sources exist.

**Permissions**: Users: root | Groups: root, exastoradm, exaadm

**Parameters**:

- `delta` (int, required): Level of redundancy by which the current redundancy should be decreased.
- `vid` (int, required): ID of an existing volume.
- `vname` (str, optional): Name of an existing volume.

**Examples**:

```bash
confd_client st_volume_decrease_redundancy {delta: 1, vname: DataVolume1}
```

## st_volume_increase_redundancy

This job increases the redundancy of the given volume by the level specified
with 'num_inc'. The new redundancy segments will be created on the nodes    
specified with 'nodes'. If no nodes have been given, the redundancy segments
will be distributed throughout the current nodes of the volume (if there is 
enough free space).

**Permissions**: Users: root | Groups: root, exastoradm, exaadm

**Parameters**:

- `delta` (int, required): Level of redundancy by which the current redundancy should be increased.
- `vid` (int, required): ID of an existing volume.
- `nodes` (list, optional): Node IDs to be used for the new redundancy.
- `vname` (str, optional): Name of an existing volume.

**Examples**:

```bash
confd_client st_volume_increase_redundancy {delta: 1, vname: DataVolume1}
```

## st_volume_set_forced_red_level

If a redundancy level is forced, every operation is redirected to the       
redundancy segment that represents that level. In case of write operations, 
no redundancy data is written (since you write directly on a redundancy     
segment that does not have other redundancy segments). This is a debug      
function, so use it with caution! If the forced redundancy level is set     
within an operation property (st_op_prop), the redundancy level will then   
overwrite the one that has been set using this function (for that particular
operation).

**Permissions**: Groups: root

**Parameters**:

- `force_red_level` (int, required): 'Forced_red Level of redundancy that should be used for reading/writing. Level 1 selects the master segment. Example: it enables the ''normal'' behaviour, including red. write operations and automatic
- `vid` (int, required): Volume ID.
- `vname` (str, optional): Name of an existing volume.

**Examples**:

```bash
confd_client st_volume_set_forced_red_level {force_red_level: 1, vname: DataVolume1}
```

## st_volume_set_io_status

While I/O for a volume is disabled, all I/O operations, like read and write,
on the volume will return error STE_AGAIN. Therefore the volume will "hang" 
for the applications.

**Permissions**: Users: root | Groups: root, exastoradm, exaadm

**Parameters**:

- `app_io` (bool, required): Application I/O in.
- `int_io` (bool, required): Internal I/O.
- `vid` (int, required): Volume ID.
- `vname` (str, optional): Volume name.

**Examples**:

```bash
confd_client st_volume_set_io_status {app_io: true, int_io: true, vname: DataVolume1}
```

## st_volume_set_ports

The contents of an archive volume can be accessed using http(s)/ftp(s)/sftp 
by configuring port numbers for the desired protocols (so that, for         
instance, the contents can be browsed using an FTP client on the configured 
ftp_port, subject to the access restrictions imposed by the volume owner and
permissions).

**Permissions**: Groups: root, exaadm, exadbadm

**Parameters**:

- `vid` (int, required): ID of an existing achive volume.
- `ftp_port` (int, optional): FTP port, on which the archive volume should be accessible.
- `ftps_port` (int, optional): FTPS port, on which the archive volume should be accessible.
- `http_port` (int, optional): HTTP port, on which the archive volume should be accessible.
- `https_port` (int, optional): HTTPS port, on which the archive volume should be accessible.
- `sftp_port` (int, optional): SFTP port, on which the archive volume should be accessible.
- `vname` (str, optional): Name of an existing volume.

**Examples**:

```bash
confd_client st_volume_set_ports {ftp_port: 23023, ftps_port: 21021, vname: ArchiveVolume1}
```

## st_volume_set_priority

Each volume has a priority value that determines two things:                

 • how many background operations are created (relative to other volumes)   
   when restoring a node or creating a snapshot                             
 • whether I/O operations should be preferred before operations from other  
   volumes. Minimum priority is 20 and maximum is 0. The default is 10.

**Permissions**: Users: root | Groups: root, exastoradm, exaadm

**Parameters**:

- `prio` (int, required): New priority.
- `vid` (int, required): ID of an existing volume.
- `vname` (str, optional): Name of an existing volume.

**Examples**:

```bash
confd_client st_volume_set_priority {prio: 1, vname: DataVolume1}
```

## st_volume_set_shared

Set/unset the 'shared' flag of a specified volume The 'shared' flag cannot
be changed if the volume is currently opened by more than one user.

**Permissions**: Users: root | Groups: root, exastoradm, exaadm

**Parameters**:

- `shared` (bool, required): Set or clear the shared flag.
- `vid` (int, required): ID of an existing volume.
- `vname` (str, optional): Name of an existing volume.

**Examples**:

```bash
confd_client st_volume_set_shared {shared: false, vname: DataVolume1}
```

## remote_volume_add

This job creates a remote volume.

**Permissions**: Users: root | Groups: root, exaadm, exastoradm

**Parameters**:

- `url` (str, required): The URL address of the remote volume.
- `vol_type` (str, required): 'The type of remote volume. The following are valid types: smb, ftp, sftp, webhdfs, webdav, file, s3, gs, azure.'
- `labels` (list, optional): Additional labels for the volume.
- `options` (list, optional): 'Options of the volume. Valid options include: verbose, noverifypeer, nocompression, noresume, forcessl, earlytls, webdav, webhdfs, delegation_token=<token>, user=<user>, timeout=<timeout>, and proxy=
- `owner` (list|tuple, optional): Tuple of (User id, group id) of the database owner in type integer or (user name, user group name).
- `password` (str, optional): The password for the remote service.
- `remote_volume_id` (str|int, optional): The remote volume ID. The ID must be over 10000 (5 digits). If you do not enter an ID, one will be created for you.
- `remote_volume_name` (str, optional): The name of the remote volume. If you do not enter a name, one will be created for you.
- `username` (str, optional): The username for the remote service.

**Examples**:

```bash
confd_client remote_volume_add {owner: [500, 500], url: 'ftp://ftpserver:12345/optional-directory/', vol_type: ftp}
```

## remote_volume_remove

This job deletes a remote volume.

**Permissions**: Users: _owner | Groups: root, exaadm, exastoradm

**Parameters**:

- `remote_volume_name` (str, required): The remote volume name.
- `remote_volume_id` (str|int, optional): The remote volume ID.

**Examples**:

```bash
confd_client remote_volume_remove {remote_volume_name: vol1}
```

## remote_volume_list

This job lists all remote volumes.

**Permissions**: Users: root | Groups: root, exaadm, exastoradm

## remote_volume_list_details

This job lists all remote volumes.

**Permissions**: Users: root | Groups: root, exaadm, exastoradm

## remote_volume_info

This job provides information about the remote volume.

**Permissions**: Users: _owner | Groups: root, exaadm, exastoradm

**Parameters**:

- `remote_volume_name` (str, required): The remote volume name.
- `remote_volume_id` (str|int, optional): The remote volume ID.

**Examples**:

```bash
confd_client remote_volume_info {remote_volume_name: vol1}
```

## remote_volume_state

This job retuns the state of the remote volume. For example, it is running.

**Permissions**: Users: _owner | Groups: root, exaadm, exastoradm

**Parameters**:

- `remote_volume_name` (str, required): The remote volume name.
- `remote_volume_id` (str|int, optional): The remote volume ID.

**Examples**:

```bash
confd_client remote_volume_state {remote_volume_name: vol1}
```

## object_volume_add

Adds a new volume to EXAStorage pointing to a cloud storage.

    owner: '500:500', prefix: Europe/France/Nouvelle-Aquitaine/Bordeaux, priority: 10,
    region: eu-central-1, vol_type: s3}

**Permissions**: Users: root | Groups: root, exaadm, exastoradm

**Parameters**:

- `name` (str, required): The name of the new data volume.
- `vol_type` (str, required): The data volume type. For example, enter S3 for AWS.
- `bucket` (str, optional): The name of an existing bucket you want to add to the data volume.
- `credentials` (str, optional): 'The type of credentials needed to access the data volume. Valid values are: root, IAM, and MFA.'
- `end_point` (str, optional): The gateway endpoint tag.
- `labels` (list, optional): A description of the new data volume.
- `options` (str, optional): The volume options.
- `owner` (list|tuple, optional): Tuple of the database owner in type integer or (user name, user group name).
- `permissions` (str, optional): The permissions for individual files within the data volume.
- `prefix` (str, optional): The prefix of the storage area within the bucket where you want to add the data volume.
- `priority` (str, optional): The priority of the volume in terms of process scheduling priority. A higher number specifies a higher priority. For most cases, you can set the priority to a value of 10.
- `region` (str, optional): The encoded geographical region that stores the cloud resources. For example, Europe (Frankfurt) is eu-central-1.
- `shared` (bool, optional): Sets whether the data volume is shared. Valid values are True and False.

**Examples**:

```bash
confd_client object_volume_add {bucket: Bucket1, credentials: root, end_point: 'Name:my-endpoint-01', name: DataVolume2,
```

## object_volume_info

This job provides information about the data volume.

**Permissions**: Users: root | Groups: root, exaadm, exastoradm

**Parameters**:

- `name` (str, required): The name of the data volume.
- `vol_id` (int, optional): The ID of the data volume.

**Examples**:

```bash
confd_client object_volume_info {name: DataVolume2}
```

## object_volume_list

This job lists all data volumes.

  {}

**Permissions**: Users: root | Groups: root, exaadm, exastoradm, exausers

## object_volume_remove

This job removes a data volume and its associated running storage service.
The data volume cannot be in use when you use this command.

**Permissions**: Users: root | Groups: root, exaadm, exastoradm

**Parameters**:

- `name` (str, required): The name of the data volume.
- `vol_id` (int, optional): The ID of the data volume.

**Examples**:

```bash
confd_client object_volume_remove {name: DataVolume2}
```
