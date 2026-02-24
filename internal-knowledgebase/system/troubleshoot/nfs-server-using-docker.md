---
tool_name: cos
doc_type: troubleshoot
category: system
title: "NFS server using Docker"
summary: "This how-to can be used in conjunction with local mount-point backups for EXAStorage file-based remote volumes."
---
# NFS server using Docker

### Overview

This how-to can be used in conjunction with local mount-point backups for EXAStorage file-based remote volumes.

### Prerequisites

Docker, internet access, root access

### Create NFS directory used for Exasol backups

```bash
mkdir /mnt/exa/nfs
```

### Setup NFS docker container

```bash
docker run -d --name nfs --net=host -p 2049:2049 --privileged -v /mnt/exa/nfs:/exa -e
SHARED_DIRECTORY=/exa itsthenetwork/nfs-server-alpine:latest
```

### Firewalld for NFS

```bash
firewall-cmd --permanent --add-service=nfs
firewall-cmd --permanent --add-service=mountd
firewall-cmd --permanent --add-service=rpc-bind
firewall-cmd --reload
```

### NFS utils

CentOS 7 has no proper NFS utils installed. NFS utils need to be installed on all hosts (use psh to install on all data nodes).

```bash
yum install nfs-utils
```

### Local NFS mount-point

Create NFS mount directory (repeat for all nodes or use cosexec -art)

```bash
mkdir -p /mnt/nfs
```

### Mount NFS share

```bash
mount -v <SERVER_IP>: /mnt/nfs/
```

### NFS for Exasol backups

Once the NFS share is mounted create a subfolder which is owned by exasolution - remember otherwise we cannot push backups to that directory.

```bash
mkdir /mnt/nfs/exasol_backups
chown exasolution.exasolution /mnt/nfs/exasol_backups
```

Adjust sdfs_remote_volumes.cfg to use NFS share

```bash
10003 file:///mnt/nfs/exasol_backups nocompression - -
```
