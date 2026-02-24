---
tool_name: confd_client
doc_type: guide
category: system
title: "SDDC: Switch Active Production Database to DR Site"
summary: "Internal failover runbook for switching active database operation from production site to DR site using confd_client and node suspension workflow."
---

# SDDC: Switch Active Production Database to DR Site

## Scope

Internal failover procedure for switching active workload from production site database to DR-site database.

## Preconditions

- Confirm failover decision and maintenance/incident approval.
- Identify active-site node IDs to suspend.
- Ensure DR database object (for example `DR_<db_name>`) is configured and ready.

## Procedure

### 1. Stop active production database

```bash
confd_client db_stop db_name: <PROD_DB_NAME>
```

### 2. Stop active-site node services

Run on all nodes of the currently active site:

```bash
systemctl --user stop c4_cloud_command
systemctl --user stop c4
```

### 3. Verify node state

From a passive/DR-side reachable node:

```bash
confd_client node_state
```

### 4. Suspend active-site nodes

```bash
confd_client node_suspend nid: '[<nid1>,<nid2>,<nid3>]'
```

### 5. Start DR database

```bash
confd_client db_start db_name: <DR_DB_NAME>
```

## Validation

- `confd_client db_state db_name: <DR_DB_NAME>` reports `running`.
- Active-site nodes remain suspended.
- Client connectivity and critical workload checks pass on DR site.

## Notes

- Keep exact node/database identifiers in incident/change record.
- Use the corresponding switch-back runbook after production site recovery.
