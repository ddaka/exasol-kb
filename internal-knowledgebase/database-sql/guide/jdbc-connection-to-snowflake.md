---
tool_name: internal-knowledgebase
doc_type: guide
category: database-sql
title: "Configure a JDBC Connection from Exasol to Snowflake"
summary: "Install the Snowflake JDBC driver in Exasol and create a JDBC connection object for IMPORT workflows or virtual schema setups."
---

# Configure a JDBC Connection from Exasol to Snowflake

## Overview

This guide describes how to configure a Snowflake JDBC connection for use from Exasol.

## Prerequisites

- Snowflake account, username, and password.
- Snowflake connection parameters (account, warehouse, database, schema, role).
- Access to ExaOperation/driver management for JDBC driver upload.

## Procedure

### 1. Download Snowflake JDBC driver

- <https://repo1.maven.org/maven2/net/snowflake/snowflake-jdbc/>

### 2. Configure driver in Exasol

Use ExaOperation JDBC driver management:

- [Add JDBC Drivers](https://docs.exasol.com/db/latest/administration/on-premise/manage_drivers/add_jdbc_driver.htm)

Driver settings:

- Main class: `net.snowflake.client.jdbc.SnowflakeDriver`
- JDBC prefix: `jdbc:snowflake:`
- Disable security manager (if required by your environment)

Upload the driver JAR and save configuration.

### 3. Create connection object in Exasol

```sql
CREATE CONNECTION SF_CON TO
'jdbc:snowflake://<account_name>.snowflakecomputing.com/?warehouse=<warehouse_name>&db=<db_name>&schema=<db_schema>&role=<user_role>&CLIENT_SESSION_KEEP_ALIVE=true'
USER '<sf_user>' IDENTIFIED BY '<pwd>';
```

Replace all `<...>` placeholders with actual values.

## Validation

- Execute a test import query via `IMPORT FROM JDBC AT SF_CON ...`.
- Confirm expected authentication and schema access.

## Notes

For full parameter details, see:

- <https://docs.snowflake.com/en/user-guide/jdbc-configure.html#connection-parameters>

## Related

- <https://github.com/bjamshid/sn-virtual-schemas>
