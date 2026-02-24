---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "INTERNAL: Synchronize Archive Volumes via FTP (Shell)"
summary: "With this simple Python script, you can easily synchronize local archive volumes between clusters. Transport is TLS encrypted (self._ftp = FTP_TLS). After volumes have been..."
---
# INTERNAL: Synchronize Archive Volumes via FTP (Shell)

## Overview

With this simple Python script, you can easily synchronize local archive volumes between clusters. Transport is TLS encrypted (self._ftp = FTP_TLS). After volumes have been initially synchronized all files added or deleted will be added or deleted in the target archive volume. This script does not support synchronizing specific days or backup IDs but it can be easily adjusted to your needs.

Run this script on each node and file you want to transfer/sync.

## Procedure

```
copy_backup_ftp_shell.py ftp://admin:admin@IP/SOURCEVOLUME ftp://admin:admin@IP/TARGETVOLUME NODEID
```
## Additional Notes

no automatic parallel transfers, please use the UDF for parallel transfers

## Additional References

See [Synchronize Archive Volumes via FTP](https://exasol.my.site.com/s/article/Synchronize-Archive-Volumes-via-FTP) for transfer via UDF.

## Downloads

[backup_copy_ftp_shell.zip](https://github.com/exasol/Internal-Knowledgebase/files/9990882/backup_copy_ftp_shell.zip)
