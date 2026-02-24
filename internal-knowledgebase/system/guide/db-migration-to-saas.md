---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "DB migration to SaaS"
summary: "Operational workflow for migrating an existing Exasol database into a SaaS customer database, including post-restore SaaS-specific steps."
---
# DB migration to SaaS

## Purpose

Migrate an existing Exasol database into a SaaS customer database and complete required SaaS post-restore configuration.

## Scope

This guide focuses on the SaaS-specific migration phase after the source backup is prepared.

## High-level workflow

1. Create a backup from the source database.
2. Create the target SaaS customer database and keep it empty.
3. Restore the backup into the target SaaS database.
4. Apply SaaS post-restore configuration.

## Prerequisites

- Source backup is available and accessible from the target environment.
- Target SaaS customer database is provisioned.
- You have access to SaaS management tooling and required secrets in AWS SSM.

## 1) Preconfigure target DB access for SaaS management

SaaS management must be able to connect to the target database as `sys`.

Use one of the following options.

### Option A: Update `sys` password in AWS SSM

Update the password value stored for the target customer database in AWS SSM Parameter Store.

### Option B: Update DB `sys` password to match SSM

```shell
dwad_client set-sys-password <db_name> <password>
confd_client db_stop <db_name>
confd_client db_start <db_name>
```

### Option C: Start DB in maintenance mode (temporary)

```shell
dwad_client start-maintenance <db_name>
```

Use maintenance mode only for controlled short windows.

## 2) Recreate required SaaS database users

Use passwords from SSM:

- `ssm://{cdf.PLATFORM_REFERENCE}/exa_saas_admin_user`
- `ssm://{cdf.PLATFORM_REFERENCE}/exa_monitoring_user`
- `ssm://{cdf.PLATFORM_REFERENCE}/exa_external_usage_user`

```sql
CREATE USER exa_saas_admin_user IDENTIFIED BY "<admin-password>";
GRANT CREATE SESSION TO exa_saas_admin_user;
GRANT DBA TO exa_saas_admin_user;

CREATE USER exa_monitoring_user IDENTIFIED BY "<monitoring-user-password>";
GRANT CREATE SESSION TO exa_monitoring_user;
GRANT SELECT ANY DICTIONARY TO exa_monitoring_user;
GRANT EXPORT TO exa_monitoring_user;

CREATE USER exa_external_usage_user IDENTIFIED BY "<external-user-password>";
GRANT CREATE SESSION TO exa_external_usage_user;
```

## 3) Reinsert SaaS user-to-database mappings

After restore, previously provisioned SaaS mappings may be missing. Identify active users for the target DB from SaaSDB and reinsert rows as needed.

```sql
WITH latest_users_to_databases AS (
  SELECT MAX(id) id
  FROM saas.users_to_databases
  GROUP BY user_uuid, db_uuid
), latest_users AS (
  SELECT MAX(id) id
  FROM users
  GROUP BY uuid, account_uuid
), active_users_to_databases AS (
  SELECT user_uuid
  FROM users_to_databases
  WHERE id IN (SELECT id FROM latest_users_to_databases)
    AND db_uuid = :{db_uuid}
    AND status != 'deleted'
), active_users AS (
  SELECT uuid
  FROM users
  WHERE id IN (SELECT id FROM latest_users)
    AND status != 'deleted'
)
SELECT users.*
FROM users
JOIN active_users ON users.uuid = active_users.uuid
JOIN active_users_to_databases ON users.uuid = active_users_to_databases.user_uuid;
```

Example insert:

```sql
INSERT INTO users_to_databases (user_uuid, db_uuid, status, roles_to_grant)
VALUES ('<user_uuid>', '<db_uuid>', 'tocreate', 'ZGJh');
```

## 4) Storage and feature post-steps

If required by your environment:

```shell
# Disable scheduled sbfs sync jobs first and ensure no sbfs process is running.
sbfs remote-format 4 -metadata-volume 0 -data-volume 3
sbfs sync-all 2 4 -next-expiring-first
```

Enable file upload feature:

```sql
INSERT INTO db_to_features (db_uuid, feature, status)
VALUES ('<db_uuid>', 'file-upload', 'todo');
```

If file upload provisioning fails, check `db_to_features`, remove stale file-upload keys/EXAConf entries, and retry.

## Validation checklist

- SaaS management can connect to target DB as expected.
- Required SaaS users exist with correct grants.
- Required rows in `users_to_databases` are present.
- File upload feature is provisioned successfully (if enabled).


