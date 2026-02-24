---
tool_name: cos
doc_type: guide
category: system
title: "Internal - Recover cluster on old COS version when license node update fails"
summary: "Emergency legacy workflow to recover cluster control plane on older COS when license-node update fails and appserver initialization is broken."
---
# Internal - Recover cluster on old COS version when license node update fails

## Scenario

Use this recovery path when all are true:

- License node update to newer COS failed.
- `appserverd` cannot reinitialize.
- EXAoperation/Zope DB is corrupted.

## Prerequisite

Data nodes are suspended and **not** shut down.

## Recovery procedure

1. Stop COS on license node (if running).
2. Edit `/etc/cos.conf` and set `COS_DIRECTORY` to known-good old version, for example:

```text
COS_DIRECTORY="/usr/opt/EXASuite-5/EXAClusterOS-5.0.14"
```

3. On one data node, start COS and verify core services (`logd`, `lockd`, `dwad`) are up.
4. Start `appserverd` in license-server mode from data node:

```shell
cosexec --single-instance --auto-restart --auto-add -- $COS_DIRECTORY/libexec/appserverd
```

5. Monitor appserver log and confirm successful cluster reinitialization.
6. Verify cluster configuration is loaded (`cosps -N`).
7. Re-check license-node `/etc/cos.conf` still points to old `COS_DIRECTORY`.
8. Reboot license node so old version/config synchronize during boot.

## Validation

- Cluster nodes visible and online in `cosps -N`.
- Core partitions/services stable.
- EXAoperation/appserver state healthy.

## Notes

- This is a legacy COS 5/6 recovery workflow.
- Perform follow-up plan for supported upgrade path after stabilization.


