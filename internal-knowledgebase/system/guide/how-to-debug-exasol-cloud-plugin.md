---
tool_name: confd_client
doc_type: guide
category: system
title: "How to debug Exasol cloud plugin"
summary: "Debug workflow for cloud plugin issues using source references, plugin inspection commands, and debug collection jobs."
---
# How to debug Exasol cloud plugin

## Purpose

Provide a practical starting workflow for investigating cloud plugin issues on Exasol cloud deployments.

## 1) Locate plugin implementation

- Cloud plugin source:
  - <https://github.com/exasol/exawolke/tree/master/src/Cloud.UIBackend>
- EXAoperation plugin framework:
  - <https://github.com/exasol/exaoperation-plugins>
- Product doc reference:
  - <https://docs.exasol.com/db/latest/administration/aws/plugin/cloud_ui_plugin.htm>

## 2) Validate plugin presence and metadata (COS)

Use `confd_client` plugin jobs (validated against `cos/confd-system-and-infrastructure.md`):

```shell
confd_client plugin_list
confd_client plugin_info {plugin_name: <plugin_name>}
```

If reinstalling or replacing plugin artifacts:

```shell
confd_client plugin_add {bucket_name: <bucket>, bucketfs_name: <bucketfs>, plugin_name: <plugin_name>}
confd_client plugin_remove {plugin_name: <plugin_name>}
```

## 3) Collect diagnostics

Capture logs and debug payloads for analysis:

```shell
confd_client log_collect {services: [ConfD, DWAd, HealthD], start_time: '<YYYY-MM-DD HH:MM:SS>', end_time: '<YYYY-MM-DD HH:MM:SS>'}
confd_client debug_collect {debug_info: Coredumps}
```

## 4) Correlate behavior

- Compare observed failure path with plugin source code flow.
- Validate expected plugin registration/state in runtime configuration.
- Attach collected diagnostics to incident/change record.

## Reference

- Internal command syntax source: `documents/cos/confd-system-and-infrastructure.md`


