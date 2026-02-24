---
tool_name: cos
doc_type: reference
category: system
title: "Internal - Push backups into SDFS using curl, wget and ssh"
summary: "Push backups into SDFS using curl, wget and ssh"
---
# Internal - Push backups into SDFS using curl, wget and ssh

## Overview

Push backups into SDFS using curl, wget and ssh

## Examples

Required files e.g. two node cluster (full backup set, status file is not required):

* id[0-999]/level_[0-9]/node_0/backup_[timestamp]
* id[0-999]/level_[0-9]/node_1/backup_[timestamp]
* id[0-999]/level_[0-9]/node[0-255]/metadata_[timestamp]

```
curl --ftp-create-dirs -T [backupfile] -u [EXAoperationUser]:[UserPassword] ftp://[Public-IP]:2021/[SDFSVolumeID]/[DBname]/[BackupID]/[Level]/[NodeID]/
```

### Upload (via curl)

Upload file and create folders if they do not exist (example)

```
curl --ftp-create-dirs -T backup_201602031455 -u admin:admin sftp://192.168.178.11:2021/v0001/exa_db1/id_883/level_0/node_1/
```

### Upload file from stdin with cat|sdfs addraw to sdfs volume

```
cat metadata_201507130400|sdfs addraw v5 - baur/id_285/level_1/node_0/metadata_201507130400
```

### Download (curl)

Download backup files from SDFS and pipe into stdin (curl parameter "-T -") (use sdfs getraw for using a pipe)

```
for i in $(sdfs shortlist v5|grep level_0);do sdfs getraw v5 $i |curl --limit-rate 5m -v --ftp-create-dirs -T - -u admin:admin ftp://213.95.157.21:2021/v0001/$i;done
```

### Download backup sets specific backup id with wget (example downloads backup id 44)

```
wget --no-parent --recursive --level=5 --user=admin --password=admin --directory-prefix=/tmp/level-0 ftp://172.30.48.11:2021/v0001/exa_sfp/id_44
```

### COPY from local SDFS into local SDFS and convert, if backup comes from compressed remote volume

```
sdfs getraw v2 baur/id_401/level_1/node_0/backup_201507130400 | convert_curl_sdfs_file.py -d -i - -o - | sdfs addraw v5 - baur/id_401/level_1/node_0/backup_201507130400
```

### COPY from compressed or uncompressed remote archive volume

```
sdfs getraw r0 baur/id_401/level_1/node_0/backup_201507130400 | sdfs addraw v5 - baur/id_401/level_1/node_0/backup_201507130400
```

### SSH transfer from a remote archive into a local archive

```
ssh -c aes128-ctr root@$NodeIP 'sdfs getraw v2 baur/id_401/level_1/node_0/backup_201507130400' |sdfs addraw v1 - baur/id_401/level_1/node_0/backup_201507130400
```

### SSH large file transfer (pv and buffer required, buffer and pv both need to be installed)

```
ssh -c aes128-ctr root@${nodeIp} 'sdfs getraw v2 baur/id_401/level_1/node_0/backup_201507130400 |mbuffer -m 1G|
```
