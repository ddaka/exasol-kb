---
tool_name: confd_client
doc_type: guide
category: system
title: "SDDC: Switch Active DR Database Back to Production Site"
summary: "Internal failback runbook for resuming production-site nodes, stopping DR database, and reactivating primary production database."
---

# SDDC: Switch Active DR Database Back to Production Site

## Scope

Internal failback procedure to switch active workload from DR database back to production database.

## Preconditions

- Production-site nodes are healthy (or can be resumed).
- Recovery/failback decision approved.
- Database names and node IDs for production site are confirmed.

## Procedure

### 1. Resume production-site nodes

```bash
confd_client node_resume nid: '[<nid1>,<nid2>,<nid3>]' enable: true
```

If nodes are already running, verify state and continue.

### 2. Start site services

Run on resumed production-site nodes:

```bash
systemctl --user start c4
systemctl --user start c4_cloud_command
```

### 3. Verify node availability

From an active node:

```bash
confd_client node_state
```

### 4. Stop DR database

```bash
confd_client db_stop db_name: <DR_DB_NAME>
```

### 5. Start production database

```bash
confd_client db_start db_name: <PROD_DB_NAME>
```

## Validation

- `confd_client db_state db_name: <PROD_DB_NAME>` is `running`.
- DR database remains stopped after failback.
- Client connections and key workload checks pass on production site.

## Notes

- Keep failback timing and node/database identifiers documented in change/incident records.
- If partial node recovery exists, validate redundancy and resource distribution before declaring completion.
