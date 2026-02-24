---
tool_name: cos
doc_type: guide
category: system
title: "How to rotate coredumps in SaaS/V8"
summary: "Configure host-level logrotate rules for Exasol coredumps to prevent root filesystem exhaustion on SaaS/V8 nodes."
---
# How to rotate coredumps in SaaS/V8

## Purpose

Prevent root filesystem exhaustion caused by accumulated core dumps.

## Scope

This procedure is executed on the host OS (outside COS namespace).

## 1) Add logrotate rule on each node

Create `/etc/logrotate.d/coredumps`:

```conf
/home/ubuntu/.ccc/play/local/<PLAY_ID>/main/<NODE>/data/spool/coredumps/core.* {
  daily
  rotate 7
  compress
  delaycompress
  missingok
  notifempty
  create 600 root root
}
```

Adjust `<PLAY_ID>` and `<NODE>` for each host.

## 2) Validate logrotate configuration

```shell
sudo logrotate -d /etc/logrotate.conf
```

## 3) Operational notes

- Scheduled rotation is executed by host cron/anacron.
- Manual cleanup path inside COS namespace is `/exa/spool/coredumps/`.

## De-duplication note

Canonical COS coredump/path references are maintained in:

- `documents/cos/cos_directory0structure.md`
- `documents/cos/confd-system-and-infrastructure.md` (`debug_collect`)

## Reference

- <https://exasol.my.site.com/s/article/Changelog-content-20772?language=en_US>


