---
tool_name: confd_client
doc_type: guide
category: system
title: "SaaS - Restart ConfD Service"
summary: "Restart ConfD on SaaS management/access nodes with controlled service-impact handling."
---
# SaaS - Restart ConfD Service

## Impact

Restarting `confd` can briefly interrupt dependent SaaS operations. Plan and communicate a short maintenance window.

## Prerequisites

- Access to SaaS production/customer AWS account.
- Container access on management or access node:
  [`saas-how-to-connect-to-the-container.md`](saas-how-to-connect-to-the-container.md)

## Procedure

1. Connect to target node container namespace.

2. Choose one restart method.

### Method A: restart by process PID

1. Find ConfD process:

```bash
ps -ef | grep -i '[c]onfd'
```

2. Kill process:

```bash
kill <CONFD_PID>
```

3. Verify it is running again:

```bash
ps -ef | grep -i '[c]onfd'
```

### Method B: restart via COS partition

1. Find ConfD partition:

```bash
cosps -N | grep -i confd
```

2. Remove partition:

```bash
cosrm -a <CONFD_PARTITION_ID>
```

3. If it does not auto-restart, relaunch binary explicitly:

```bash
cosexec -a <PATH_TO_CONFD_BINARY> -v
```

## Validation

- `confd` process/partition is present after restart.
- Expected SaaS control operations respond normally.

## Canonical references

- `documents/cos/cos-system-partitions.md` (confd partition behavior)
