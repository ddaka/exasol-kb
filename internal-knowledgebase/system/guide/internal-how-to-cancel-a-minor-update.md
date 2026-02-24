---
tool_name: cos
doc_type: guide
category: system
title: "Internal - Cancel a minor update"
summary: "Cancel a queued minor update before license server restart in legacy EXAoperation-based environments."
---
# Internal - Cancel a minor update

## Scope

Use this procedure only when update package is uploaded but license server has **not** been restarted yet.

## Procedure

1. On license server, restore normal startup target:

```shell
chkconfig --del update_cos
chkconfig --add cos
```

2. Remove uploaded update package from EXASuite path (example legacy path):

```text
/usr/opt/EXASuite-6/EXAClusterOS-6.x.x
```

3. Restart EXAoperation so pending update message disappears from Software tab.

## Notes

- This is a legacy EXAoperation workflow.
- For modern update handling, prefer canonical system update commands/workflows.

## De-duplication note

Canonical update command reference:

- `documents/cos/confd-system-and-infrastructure.md` (`update_system`)


