---
tool_name: internal-knowledgebase
doc_type: guide
category: database-sql
title: "Restrict User Data Visibility in EXA_ALL_* Dictionaries"
summary: "Enable restrictUserInfoInExaAllSysTables to limit cross-user metadata visibility in EXA_ALL_USERS and EXA_ALL_SESSIONS for multi-tenant environments."
---

# Restrict User Data Visibility in EXA_ALL_* Dictionaries

## Overview

In multi-tenant environments, dictionary visibility may need to be restricted so users only see their own metadata.

From Exasol `7.1.27`, parameter `restrictUserInfoInExaAllSysTables` can be used for this purpose.

## Important Considerations

- Feature was introduced as internal/limited-support behavior; validate against your support policy before rollout.
- Take a full backup before applying startup-parameter changes.
- Existing queries or tools relying on full `EXA_ALL_*` visibility may break.

## Prerequisites

- DBA-level access to stop/start database and modify startup parameters.
- Privileges to adjust grants (optional hardening step).

## Activation Procedure

1. Stop database.
   - <https://docs.exasol.com/db/latest/administration/on-premise/manage_database/stop_db.htm>

2. Add startup parameters:

```text
-restrictUserInfoInExaAllSysTables=1
-createSystemCatalog=1
```

- <https://docs.exasol.com/db/latest/administration/on-premise/manage_database/add_db_parameters.htm>

3. Start database.
   - <https://docs.exasol.com/db/latest/administration/on-premise/manage_database/start_db.htm>

4. Validate dictionary behavior.
   - `EXA_ALL_USERS` and `EXA_ALL_SESSIONS` should show only user-relevant rows.

5. Remove one-time catalog rebuild parameter after successful startup:

```text
-createSystemCatalog=1
```

- <https://docs.exasol.com/db/latest/administration/on-premise/manage_database/remove_db_parameters.htm>

## Optional Hardening

Restrict schema discovery for non-privileged users:

```sql
REVOKE USE ANY SCHEMA FROM PUBLIC;
```

Then grant explicit schema usage as needed:

```sql
GRANT USAGE ON my_schema TO my_user;
```

## References

- <https://exasol.atlassian.net/browse/SPOT-19134>
- <https://docs.exasol.com/db/latest/administration/on-premise/manage_db.htm>
- <https://docs.exasol.com/db/latest/database_concepts/privileges.htm>
