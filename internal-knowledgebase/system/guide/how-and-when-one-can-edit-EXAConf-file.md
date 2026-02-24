---
tool_name: confd_client
doc_type: guide
category: system
title: "How and when to edit EXAConf"
summary: "Guidance on when manual EXAConf editing is allowed and how to apply changes safely in COS environments."
---
# How and when to edit EXAConf

## Purpose

Define when manual `EXAConf` edits are acceptable and provide a safe procedure to apply and commit those edits.

## Background

For Exasol 8, configuration should normally be managed via ConfD (`confd_client` / REST API). Most settings are stored in `/exa/etc/EXAConf`, and ConfD writes changes there while retaining historical copies (`EXAConf.0`, `EXAConf.1`, ...).

Manual edits are allowed only for settings not exposed through public/documented interfaces.

## When manual editing is allowed

Manual editing is allowed only when all conditions are true:

- Required setting is not manageable through public ConfD/API interfaces.
- Change was confirmed by Development or a qualified Support engineer.
- Change window and rollback path are defined.

Reference for public interfaces:

- <https://docs.exasol.com/db/latest/home.htm>

## Prerequisites

- Shell access to COS with sufficient privileges.
- Backup/copy of current `/exa/etc/EXAConf`.
- Planned validation after commit.

## Manual edit procedure

1. Open `/exa/etc/EXAConf` in an editor.
2. Apply required changes.
3. Set checksum to `COMMIT`:

```shell
sed -i '/Checksum =/c\ Checksum = COMMIT' /exa/etc/EXAConf
```

4. Synchronize and commit configuration:

```shell
exaconf commit
```

Expected output includes successful synchronization and local commit steps.

## Common failure mode

If `Checksum` is not set to `COMMIT`, subsequent commands may fail with misleading errors (for example authentication-related errors from ConfD).

## Reference

- ConfD documentation: <https://docs.exasol.com/db/latest/confd/confd.htm>


