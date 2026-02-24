---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Fix a corrupt RPM database"
summary: "Recover a corrupted RPM database after interrupted OS package operations."
---
# Fix a corrupt RPM database

## Problem

OS package commands fail because the RPM database is corrupted, commonly after an interrupted/rebooted update process.

Typical errors include `DB_RUNRECOVERY` and failure to open `/var/lib/rpm` package indexes.

## Prerequisites

- Root shell on the affected node.
- Correct target namespace/chroot if applicable.
- Maintenance window or approved change context.

## Symptoms

Examples:

```shell
yum clean
rpm -qa
```

Both may fail with RPM DB errors such as:

- `DB_RUNRECOVERY: Fatal error, run database recovery`
- `cannot open Packages database in /var/lib/rpm`

## Recovery procedure

1. Remove stale RPM DB lock/environment files:

```shell
rm -f /var/lib/rpm/__*
```

1. Rebuild the RPM database:

```shell
rpm --rebuilddb
```

1. Validate package DB usability:

```shell
rpm -qa | wc -l
```

If commands return successfully, RPM DB recovery is complete.

## Optional follow-up

If an OS update failed previously, rerun the failed update workflow according to your platform version and maintenance procedure.

## Result

The incident is resolved when package-manager commands execute normally and pending update operations can continue.

---

_We welcome feedback on this troubleshooting article._
