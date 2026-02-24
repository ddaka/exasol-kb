---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "Sync SaaS Environment with Datadog"
summary: "Run the SaaS monitoring sync module to reconcile Datadog dashboards/monitors with current environment state."
---
# Sync SaaS Environment with Datadog

## Purpose

Recover Datadog consistency when monitor/dashboard creation or cleanup failed during normal event processing.

The sync module reconciles:
- Dashboards
- Infra monitors
- State monitors
- Availability monitors
- Status monitors
- Leftover monitor cleanup

## Prerequisites

- Access to SaaS production AWS account.
- Instance and container access:
  - [`saas-how-to-connect-to-the-instance.md`](saas-how-to-connect-to-the-instance.md)
  - [`saas-how-to-connect-to-the-container.md`](saas-how-to-connect-to-the-container.md)

## Procedure

1. Connect to management node instance.
2. Enter container namespace.
3. Go to plugin directory:

```bash
cd /exa/data/bucketfs/bfsdefault/default/
```

4. Run sync module:

```bash
PYTHONPATH=$COS_DIRECTORY/lib python3 plugins\:sync.py
```

## Notes

- Use Python 3.
- Re-run only when needed; verify Datadog state after each execution.

## Related troubleshooting

- `documents/internal-knowledgebase/system/troubleshoot/saas-plugin-health-check-monitor.md`
- `documents/internal-knowledgebase/system/troubleshoot/change-monitors-threshold-on-saas-monitoring-plugin.md`
