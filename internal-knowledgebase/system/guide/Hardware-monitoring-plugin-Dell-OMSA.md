---
tool_name: cos
doc_type: guide
category: system
title: "Hardware monitoring plugin: Dell OMSA"
summary: "Install and enable the Dell OpenManage plugin for Exasol hardware monitoring, including optional check_omsa tooling."
---
# Hardware monitoring plugin: Dell OMSA

## Scope

Applies to Exasol environments where Dell OMSA-based hardware monitoring plugin is required.

## Plugin package

Expected naming pattern:

```text
Plugin.Administration.DELL-OpenManage-X.Y.Z-<build>.pkg
```

## 1) Upload plugin package

Upload via EXAoperation:

- `Software` -> `Software Update File`

(Use same upload workflow as regular software updates.)

## 2) Verify distribution to nodes

On license server:

```shell
psh ls -l /usr/opt/EXAplugins/
```

## 3) Install/start plugin outside COS

Run installation/start script in COS namespace on each relevant node:

```shell
ssh localhost -p20
/usr/opt/EXAplugins/Administration.DELL-OpenManage-9.x/exaoperation-gate/install_and_start
```

## 4) Enable auto-start after reboot

Add restart lines to `/etc/rc.local_cos` on each node:

```shell
ssh -p20 localhost /usr/opt/EXAplugins/Administration.DELL-OpenManage-9.x/exaoperation-gate/stop
ssh -p20 localhost /usr/opt/EXAplugins/Administration.DELL-OpenManage-9.x/exaoperation-gate/start
```

## 5) Optional: install `check_omsa`

```shell
mkdir /opt/check_omsa
tar xf check_openmanage-3.7.12.tar.gz -C /opt/check_omsa/
cos_sync_files /opt/check_omsa/
```

If license server is virtual and local install is not required:

```shell
rm -r /opt/check_omsa/
```

## Validation

- OMSA plugin process is running on target nodes.
- Hardware checks report expected output.
- Plugin restarts correctly after node reboot.


