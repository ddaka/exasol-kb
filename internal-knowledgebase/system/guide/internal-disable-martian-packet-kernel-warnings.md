---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "Disable Martian Packet Kernel Warnings (Internal)"
summary: "Procedure to disable persistent martian packet kernel warnings on legacy EXAoperation deployments when network conditions generate excessive false-positive logs."
---

# Disable Martian Packet Kernel Warnings (Internal)

## Overview

Some compliance frameworks require logging of martian packets (packets received on an unexpected interface). In some customer environments this produces persistent noise and reduces signal quality in kernel logs.

This guide explains how to disable martian packet logging on legacy EXAoperation-based systems when that operational trade-off is required.

## Prerequisites

Validate `/etc/rc.local.cos` consistency across all nodes before making changes.

Check checksum consistency if the file exists:

```text
cluster1 [root@n0010 ~]# [[ -e /etc/rc.local.cos ]] && sha256sum /etc/rc.local.cos | psh 'sha256sum -c 2>&1'
0011: /etc/rc.local.cos: OK
0012: /etc/rc.local.cos: FAILED
0012: sha256sum: WARNING: 1 computed checksum did NOT match
0013: sha256sum: /etc/rc.local.cos: No such file or directory
0013: /etc/rc.local.cos: FAILED open or read
0013: sha256sum: WARNING: 1 listed file could not be read
```

Check that the file does not exist unexpectedly on only some nodes:

```text
cluster1 [root@n0010 ~]# [[ ! -e /etc/rc.local.cos ]] && psh '[[ ! -e /etc/rc.local.cos ]] || echo "ERROR, file exists"'
0011: ERROR, file exists
0014: ERROR, file exists
```

If inconsistencies are found, resolve file ownership and content differences before applying this procedure.

## Procedure

1. On the license node, add the following lines to `/etc/rc.local.cos`:

```shell
sysctl -e net.ipv4.conf.all.log_martians=0
sysctl -e net.ipv4.conf.default.log_martians=0
```

2. Synchronize the updated file to all nodes:

```shell
cos_sync_files /etc/rc.local.cos
```

3. Apply the setting immediately on all nodes:

```shell
cosexec -a sysctl -e net.ipv4.conf.all.log_martians=0
cosexec -a sysctl -e net.ipv4.conf.default.log_martians=0
```

## Verification

Confirm both parameters are set to `0` on all nodes:

```shell
cosexec -a sysctl net.ipv4.conf.all.log_martians net.ipv4.conf.default.log_martians
```

## Notes

- This change reduces log noise but may remove useful spoofing indicators.
- Apply only when required and documented in the incident or customer change context.
