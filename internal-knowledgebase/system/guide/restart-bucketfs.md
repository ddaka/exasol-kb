---
tool_name: cos
doc_type: guide
category: system
title: "Restarting bucketfs in v8"
summary: "Minimal operational wrapper for restarting an unresponsive BucketFS service; canonical details live in COS docs."
---
# Restarting bucketfs in v8

## De-duplication status

This topic is already covered in canonical COS documentation. Keep this page as a short internal pointer only.

## Procedure

1. Restart the BucketFS service using the canonical command:

```bash
dwacli bucketfs restart bfsdefault
```

2. Verify process health:

```bash
cosps -N | grep -i bucketfs
```

3. If service name is unknown, list services first:

```bash
confd_client bucketfs_list
```

## Canonical references

- `documents/cos/confd-bucketfs.md`
- `documents/cos/cos-troubleshooting.md`
- `documents/cos/cos-system-partitions.md`
