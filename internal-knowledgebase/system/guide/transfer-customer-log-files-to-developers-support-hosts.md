---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "Transfer customer log files to developers support hosts"
summary: "Collect support logs and upload them to the developer upload portal from support hosts."
---
# Transfer customer log files to developers support hosts

## Purpose

Collect logs on customer/support hosts and transfer archives to the developer upload endpoint.

## Prerequisites

Before log collection, verify available disk space and estimate archive size:

```bash
df -h
get_support_info -s 2017-05-04 -t 2017-05-04 -e database_name -x1 -d1,2 -m
```

Example:

```text
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda2        50G   27G   20G  58% /
tmpfs           1.9G     0  1.9G   0% /dev/shm
/dev/vda1       243M   60M  170M  27% /boot
/dev/vda4        41G   16G   24G  41% /usr/opt
1972.582 MB
```

## Step 1: collect logs

- Legacy versions: `get_support_info`
- Exasol v8+: `exasupport` (preferred)

## Step 2: upload logs to developer portal

Use `upload-file.py` from support host:

```bash
upload-file.py -h
usage: upload-file.py [-h] -l LINK -f FILE [-t] [-d TEMPDIR]
```

Expected destination pattern:

```text
/srv/bugtrack/US/<TICKET_ID>/<filename>.tar.gz
```

## Related references

- `documents/internal-knowledgebase/system/troubleshoot/files-transfer-in-exasol.md`
- `documents/internal-knowledgebase/system/troubleshoot/log-collection-101.md`
