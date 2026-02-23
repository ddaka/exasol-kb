---
tool_name: confd_client
doc_type: guide
category: database-management
subcommands:
  - db_configure
technical_entities:
  - auditing
  - database
summary: >
  How to enable and disable auditing on an Exasol database via confd_client
  db_configure — captures session and SQL statement details.
---

# Enable / Disable Auditing

Auditing captures details for every session and SQL statement. Enabled by
default on new databases.

## Enable

```bash
confd_client db_configure db_name: MY_DATABASE enable_auditing: true
```

## Disable

```bash
confd_client db_configure db_name: MY_DATABASE enable_auditing: false
```

> Disabling auditing does not remove existing audit data. Use
> `TRUNCATE AUDIT LOGS` to clear it.

Both operations require the database to be stopped first, then restarted after.
