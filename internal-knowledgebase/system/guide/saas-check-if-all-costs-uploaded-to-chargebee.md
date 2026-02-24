---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "SaaS: Verify Cost Upload Completion to Chargebee"
summary: "Run a reconciliation query to compare uploaded cost reference count with eligible internal usage rows and confirm Chargebee cost sync completeness."
---

# SaaS: Verify Cost Upload Completion to Chargebee

## Overview

Use this check to confirm whether SaaS cost/usage records have been fully uploaded to Chargebee.

## Prerequisites

- Access to SaaS production environment.
- Read access to billing/usage reference tables.

## Verification Query

```sql
SELECT
    (SELECT COUNT(*) FROM costs_upload_references) AS done,
    (
      SELECT COUNT(*)
      FROM "SAAS"."INTERNAL_USAGE_DATA"
      WHERE usage_amount > 0
        AND cluster_uuid <> 'NA'
        AND usage_start_date >= (SELECT MIN(created_at) FROM customers)
    ) AS total;
```

## Interpretation

- If `done = total`, all eligible costs are uploaded.
- If `done < total`, investigate scheduler/backlog/failed upload records.

## Follow-up

- Review recent scheduler runs and error logs.
- Re-run reconciliation after remediation.
