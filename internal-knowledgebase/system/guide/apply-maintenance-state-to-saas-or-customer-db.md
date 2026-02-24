---
tool_name: confd_client
doc_type: guide
category: system
title: "Apply maintenance state to SaaS or customer DB"
summary: "Put SaaS platform or specific customer database into maintenance state for controlled operations."
---
# Apply maintenance state to SaaS or customer DB

## Purpose

Temporarily block user/API operations during maintenance windows.

## 1) SaaS-wide maintenance state

Apply maintenance behavior via CloudFront/service routing configuration in AWS.

Operationally:

- Switch CloudFront behavior to maintenance profile.
- Confirm maintenance page/response is active.

## 2) Customer DB maintenance state

Set maintenance status for a specific account/cluster/database via `confd_client`:

```shell
confd_client -c saas_cluster_update -a '{"account_uuid":"<account_uuid>","cluster_uuid":"<cluster_uuid>","db_uuid":"<db_uuid>","status":"maintenance"}'
```

## Validation

- UI operations are blocked for target scope.
- API operations are blocked for target scope.
- Status reflects maintenance mode in control plane.

## Exit maintenance

Set target status back to normal/active using corresponding update command or operational workflow.


