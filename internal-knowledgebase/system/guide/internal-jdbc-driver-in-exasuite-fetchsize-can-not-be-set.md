---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "INTERNAL: EXASuite JDBC Driver FETCHSIZE Cannot Be Set in GUI"
summary: "Internal procedure for adjusting JDBC driver FETCHSIZE via settings.cfg when EXAoperation does not expose the parameter for user-uploaded drivers."
---

# INTERNAL: EXASuite JDBC Driver FETCHSIZE Cannot Be Set in GUI

## Scope

In some integrations (for example ExaLoader with specific JDBC targets), fetch size tuning is required, but EXAoperation GUI does not expose `FETCHSIZE` for user-uploaded drivers.

## Symptom

- Desired `FETCHSIZE` cannot be configured through EXAoperation driver settings.

## Procedure

Perform this only with system support oversight.

1. Locate JDBC driver package on license node (example path):

```bash
/usr/opt/EXASuite-X/EXAClusterOS-X.Y.Z/var/jdbc/jdbc1/jdbc0001.jar
```

2. Edit adjacent `settings.cfg` and set `FETCHSIZE`.
3. Synchronize updated file to cluster nodes:

```bash
cos_sync_files <path-to-settings.cfg>
```

## Important Behavior

Driver-side fetch size handling uses an internal scale factor:

- Effective fetch size = configured `FETCHSIZE / 100`

Example from logs:

```text
2021-11-25 08:46:37.908 debu: FetchSizeData=100000
2021-11-25 08:46:37.908 INFO: Setting fetchsize to 1000
```

## Validation

- Confirm updated `settings.cfg` is present on all required nodes.
- Verify effective fetch size in runtime logs.
- Re-test affected import/export workload.
