---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: database-sql
title: "How to restart DRS properly"
summary: "Safe restart procedure for DRS when repository locks remain after crash/failover conditions."
---
# How to restart DRS properly

## Problem

After crash/failover events, DRS may fail to become active because stale repository lock sessions are still visible.

Typical symptoms:

- Repository status remains `standby` instead of `active`/`online`.
- Logs repeatedly show failed repository lock acquisition.

## Prerequisites

- Shell access to DRS service host.
- DRS web UI admin access.
- SQL access to repository main and mirror databases.
- Privileges to query and kill sessions (`SELECT ANY DICTIONARY`, `KILL ANY SESSION`, or equivalent via service user views).

## Step 1: Stop DRS

Stop service only when replication is idle or already unhealthy.

```shell
data-replication-service/stop.sh
```

## Step 2: Remove leftover DRS sessions

On repository main DB, find service sessions:

```sql
SELECT session_id, user_name, client, login_time
FROM SYS.exa_dba_sessions
WHERE client LIKE 'DataReplicationService%'
  AND session_id != current_session;
```

If connected as service user, use `SYS.exa_user_sessions` instead.

Kill stale sessions:

```sql
KILL SESSION <session_id>;
```

Repeat until no stale rows remain.

Repeat the same procedure on repository mirror DB.

## Step 2b: DRS 2.1.0 only (startup timeout workaround)

DRS 2.1.0 can abort when metadata fetch exceeds 30 seconds during sync-pool enable.

Before restarting, disable enabled pools in repository main:

```sql
SELECT * FROM SYNC_REPOSITORY.SYNC_POOL;

UPDATE sync_repository.sync_pool
   SET pool_status = 'DISABLED'
 WHERE POOL_ID = <pool_id>;
```

Record which pools were enabled so they can be restored later.

## Step 3: Start DRS

```shell
data-replication-service/start.sh
```

Wait 30-60 seconds and validate UI state:

- Main repository: `active`
- Mirror repository: `online` (if configured)

## Step 3b: Re-enable pools (if disabled in Step 2b)

Re-enable pools from UI (`drsadmin`) or repository process.

For DRS 2.1.0, enable carefully if metadata-fetch cycles are near timeout; repeated crashes can occur if overlap exceeds the internal 30-second window.

## Validation

- No repeating lock-acquisition warnings in logs.
- Repository states stable.
- Replication jobs resume as expected.

## Note

DRS 2.1.1 (released October 2024) addresses the startup timeout behavior in 2.1.0.

---

_We welcome feedback on this troubleshooting article._
