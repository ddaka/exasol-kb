---
tool_name: confd_client
doc_type: guide
category: system
title: "How to access backup files from a local archive volume via FTP (v8)"
summary: "Expose a local archive volume over FTP in Exasol v8, validate access, and optionally register it as a remote volume target."
---
# How to access backup files from a local archive volume via FTP (v8)

## Purpose

Allow controlled FTP access to backup files stored on a local archive volume in Exasol v8.

## Prerequisites

- COS access with required privileges.
- Existing archive volume, or permission to create one.
- FTP client (`lftp`) for validation.

## 1) Create archive volume with FTP port (or enable FTP on existing volume)

Create new volume:

```shell
confd_client st_volume_create {name: new_archive, disk: default, type: archive, size: '10 GiB', nodes: [11], redundancy: 1, owner: [500,500], ftp_port: 2021}
```

Enable FTP on existing archive volume:

```shell
confd_client st_volume_set_ports {vname: new_archive, ftp_port: 2021}
```

## 2) Enable login for FTP user and set password

```shell
confd_client user_modify {username: exadefusr, login_enabled: true}
confd_client user_passwd {username: exadefusr, password: new_password}
```

## 3) Validate FTP access

```shell
lftp -u exadefusr,new_password ftp://<node_ip>:2021
```

Expected: archive volume folder is visible in FTP listing.

## 4) (Optional) Register FTP endpoint as remote volume

If using it as backup target, add remote volume with `nocompression` option to avoid compression conflicts:

```shell
confd_client remote_volume_add {vol_type: ftp, url: 'ftp://<node_ip>:2021/new_archive/', owner: [500,500], username: exadefusr, password: new_password, options: [nocompression]}
```

## 5) Validate through SDFS

```shell
sdfs check-connectivity <remote_volume_name_or_id>
sdfs list <remote_volume_name_or_id>
```

## Notes

- FTP is unencrypted. Prefer FTPS/SFTP where policy requires encrypted transport.
- Command syntax validated against:
  - `documents/cos/confd-volume-management.md`
  - `documents/cos/confd-users-and-groups.md`

## Reference

- <https://docs.exasol.com/db/latest/confd/jobs/st_volume_create.htm>


