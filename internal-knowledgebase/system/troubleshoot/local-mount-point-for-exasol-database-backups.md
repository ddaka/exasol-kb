---
tool_name: cos
doc_type: troubleshoot
category: system
title: "Local mount-point for Exasol database backups"
summary: "Most san systems often offer a direct smb access method (netapp, etc). What vendors do these instructions apply to?"
---
# Local mount-point for Exasol database backups

### FAQ

Most san systems often offer a direct smb access method (netapp, etc).  What vendors do these instructions apply to?

-> They apply to any 3rd party backup solution that needs a local mount-point in order to retrieve data. This is for those vendors which cannot provide SMB, FTP, WebHDFS, S3, GCS or Azure Blob

BoostFS lesson-learned

-> During a recent POC we found out that BoostFS has a default user named FTP01 in order to allow the Exasol user exasolution to write backups into BoostFS directories add 'option *allow*-*others*' in BoostFS agent config otherwise backups won't work.

### Overview

3rd party software providers like EMC BoostFS or IBM Tivoli need local mount-points in order to work. Most of the times those tools are used to create backups of applications. Customers and prospects know these tools from other database vendors (which use filesystem based storage) and that is why they expect those tools to work with Exasol too. Furthermore when using 3rd party backup solutions backups should be written uncompressed otherwise the provided compression methods might be less effective and backups need to be decompressed on-the-fly for a backup restore (SDFS built-in LZ4 compression!).

### Prerequisites

Admin access to EXAoperation, SSH root access.

### Resolution

Due to security reasons EXAoperation does not allow file locations for Remote Volumes e.g. 'file:///'. In order to overcome this mechanism we first create a smb remote volume which is then modified under the hood. Once this is done we will create the mount-point. CAUTION: The mount-point is not configured in fstab as it might prevent the node from booting in case the storage manager is not available instead we will use /etc/rc.local_cos.

### EXAStorage Remote Volume

Add the remote volume in EXAStorage. Provide any user and password they won't be used at all. Required parameters: 'nocompression'. In that example we use protocol ftp and the name of the OS mount-point is 'testlocalbackup'. The name has to be same as the mount-point. The volume will be shown as offline.

![](images/Screenshot-2021-02-23-at-0.04.42.png)

### Manipulate sdfs_remote_volumes.cfg

Now we need to edit the remote volume config and replace '**ftp://'** with '**file:///'** and replace admin and password with '-' and '-'. Save the file with ':wq!'.

Old:

```bash
cluster50 [root@n0010 mnt]# cat /etc/cos/sdfs_remote_volumes.cfg
10003 ftp://testlocalbackup nocompression YWRtaW4= -
```

New: Specify the full path!

```bash
cluster50 [root@n0010 testlocalbackup]# cat /etc/cos/sdfs_remote_volumes.cfg
10003 file:///mnt/testlocalbackup nocompression - -
```

### Sync sdfs_remote_volumes.cfg to all cluster nodes

```bash
cluster50 [root@n0010 mnt]# cos_sync_files /etc/cos/sdfs_remote_volumes.cfg
Tue Feb 23 10:11:36 CET 2021: start script '/usr/opt/EXASuite-7/EXAClusterOS-7.0.5/sbin/cos_sync_files'.
Tue Feb 23 10:11:36 CET 2021: pack files for sync.
etc/cos/sdfs_remote_volumes.cfg
Tue Feb 23 10:11:36 CET 2021: copy to nodes n0011.c0001.exacluster.local n0012.c0001.exacluster.local n0013.c0001.exacluster.local n0014.c0001.exacluster.local n0015.c0001.exacluster.local
Tue Feb 23 10:11:37 CET 2021: script '/usr/opt/EXASuite-7/EXAClusterOS-7.0.5/sbin/cos_sync_files' finished.
```

### Create mount-points

In this example, we create local directories to store the backup. The owner of the directories needs to be **exasolution** otherwise we cannot write backups into those directories. The directories need to exist on all cluster members - also the mgmt. node!

```bash
cluster50 [root@n0010 ~]# cosexec -art mkdir -p /mnt/testlocalbackup
cluster50 [root@n0010 ~]# cosexec -art chown -R exasolution.exasolution /mnt/testlocalbackup
```
If everything is set up properly the remote volume is shown as online in EXAStorage and you can try to create a backup

### Debugging

Create a temporary login Shell for the user exasolution to simulate access problems.

```bash
usermod --shell /bin/bash exasolution
su - exasolution
```

 Revoke login Shell

```bash
usermod --shell none exasolution
```

### Automount: fstab vs. rc.local_cos

We strongly discourage using **fstab** as it might prevent the server from booting up properly, if the mount point or mount target is not available. Please create an **rc.local_cos** on each of the nodes and put the mount command there.

### Downsides

* Only uncompressed backups can be written

* Only blocking restore mode supported

* if the remote archive volume is changed in EXAoperation the config file on the OS will be overwritten and the BoostFS config needs to be recreated
