---
tool_name: internal-knowledgebase
doc_type: guide
category: database-sql
title: "Migrate a MySQL Database to Exasol"
summary: "Step-by-step migration guide for moving schema and data from MySQL to Exasol using JDBC and database_migration scripts."
---

# Migrate a MySQL Database to Exasol

## Overview

This guide describes a practical migration flow from MySQL to Exasol:

- Prepare connectivity and migration assets.
- Configure JDBC access to MySQL from Exasol.
- Generate and execute migration SQL.
- Run post-processing for datatype and key optimization.

## Prerequisites

- MySQL connection details:
  - Host or IP
  - Port
  - Database name
  - Username and password
- Exasol connection details:
  - Host or IP
  - Port
  - Target schema
  - Username and password
- Rights model for target environment (users, roles, privileges).
- MySQL JDBC driver package available in Exasol.
- Migration scripts available:
  - [`mysql_to_exasol.sql`](https://github.com/exasol/database-migration/blob/master/mysql_to_exasol.sql)
  - [`set_primary_keys.sql`](https://github.com/exasol/database-migration/blob/master/post_load_optimization/set_primary_keys.sql)
  - [`convert_datatypes.sql`](https://github.com/exasol/database-migration/blob/master/post_load_optimization/convert_datatypes.sql)

## Procedure

### 1. Configure the MySQL JDBC driver

Upload and configure the MySQL JDBC driver in your Exasol environment (legacy EXAoperation path: `Software -> JDBC Drivers`).

### 2. Create a MySQL connection object in Exasol

```sql
CREATE OR REPLACE CONNECTION MYSQL_JDBC TO
'jdbc:mysql://192.168.178.95:3306/demo'
USER 'thri' IDENTIFIED BY 'mYsql_1234';
```

Replace values with your environment-specific parameters.

### 3. Validate JDBC connectivity

```sql
SELECT *
FROM (
  IMPORT FROM JDBC AT MYSQL_JDBC
  STATEMENT 'select "Successfully connected to MySQL!" MYSQL_CONNECTION'
);
```

### 4. Generate migration commands

Run `MYSQL_TO_EXASOL` to generate DDL/DML statements:

```sql
EXECUTE SCRIPT database_migration.MYSQL_TO_EXASOL(
  'MYSQL_JDBC', -- connection name
  true,         -- case-insensitive identifiers (stored uppercase)
  'demo',       -- schema filter
  '%'           -- table filter
);
```

### 5. Execute generated SQL

Apply the generated statements in Exasol to:

- Create schemas and tables.
- Import MySQL data into Exasol.

Duration depends on data volume and network throughput.

## Post-Processing

### 1. Optimize datatypes

Run in simulation mode first:

```sql
EXECUTE SCRIPT database_migration.CONVERT_DATATYPES(
  'DEMO', -- schema filter
  '%',    -- table filter
  false   -- simulation mode
);
```

After review, apply suggested conversions with `true` as the third argument.

### 2. Add primary and foreign keys

```sql
EXECUTE SCRIPT DATABASE_MIGRATION.SET_PRIMARY_AND_FOREIGN_KEYS(
  'JDBC',
  'MYSQL_JDBC',
  'MYSQL',
  'd%',
  '%',
  'ENABLE',
  'true'
);
```

This step restores key constraints that are not automatically migrated with base table load.

## Validation Checklist

- Row counts match between source and target tables.
- Critical queries return expected results.
- Primary and foreign keys are created as intended.
- Datatype optimization output is reviewed and applied where appropriate.
- Permissions are validated for application users.

## References

- <https://github.com/exasol/database-migration>
- <https://dev.mysql.com/downloads/connector/j/>
