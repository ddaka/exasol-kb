---
tool_name: cos
doc_type: guide
category: system
title: "Cloud Monitoring (Proust) - manual data collection for the previous periods"
summary: "Temporarily backfill historical monitoring data by adding a bounded duration window to Proust collection."
---
# Cloud Monitoring (Proust) - manual data collection for the previous periods

## Purpose

By default, when a new cluster is onboarded into Cloud Monitoring (Proust), data collection begins from the date of onboarding.
Use this runbook to backfill older data for a limited period when historical dashboards are required.

## Prerequisites

- Cluster is already onboarded in Proust.
- Operator has COS access and permission to edit `/usr/local/bin/check_db.sh`.
- Defined backfill window in hours (example: `9000h` for about 12 months).

## Procedure

1. Log in to COS and open `/usr/local/bin/check_db.sh`.

2. Locate the `check_sqlquery` invocation and add `-duration "<HOURS>h"`:

```bash
...
cosexec -l "DB_NAME_stats" -s -w -r /usr/local/bin/check_sqlquery -query stats -id "DB_NAME" -timeout "360" -duration "9000h" | sed 's/^[0-9]*: *//'
...
```

3. Save the file and sync it:

```bash
cos_sync_files check_db.sh
```

4. Reset Proust state to force reload from the new duration window:

```bash
psh rm -rf /var/monitoring_exa_uat.timestamp
rm -rf /var/monitoring_exa_uat.timestamp
coskillall proust
```

5. Wait for the next collection cycle (typically every 30 minutes), then validate in Grafana:

[Health Check Dashboard](https://grafana.harvester-int.exasol.com/d/a17ec856-2552-47c2-b4a1-3e239625e75c/healthcheck?orgId=1)

6. After backfill finishes, revert `check_db.sh` by removing `-duration`, then sync again:

```bash
cos_sync_files check_db.sh
```

7. Confirm current-period monitoring continues normally.

## Operational notes

- Keep `-duration` temporary. Leaving it enabled can cause repeated heavy historical pulls.
- For safety, create a backup before editing:

```bash
cp /usr/local/bin/check_db.sh /usr/local/bin/check_db.sh.bak.$(date +%Y%m%d%H%M%S)
```

## De-duplication note

Use COS operational conventions from:

- `documents/cos/cos-troubleshooting.md`
- `documents/cos/cos_directory0structure.md`
