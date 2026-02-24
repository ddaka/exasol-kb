---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: database-sql
title: "Virtual schema cannot be created"
summary: "Troubleshooting checklist for CREATE VIRTUAL SCHEMA failures caused by driver, BucketFS, connection, or source support issues."
---
# Virtual schema cannot be created

## Problem

`CREATE VIRTUAL SCHEMA` fails and external source cannot be accessed.

## Common causes

- Adapter or JDBC driver missing in BucketFS.
- Incorrect adapter/JAR version.
- BucketFS access issue.
- Invalid connection credentials or blocked network path.
- Unsupported source dialect/adapter combination.

## Troubleshooting checklist

### 1. Verify driver artifacts in BucketFS

- Confirm adapter script JAR is uploaded.
- Confirm required JDBC driver is uploaded (if adapter needs it).
- Confirm expected version is present (not deprecated/incompatible).

### 2. Verify BucketFS access

Example check:

```shell
curl http://r:<read_password>@<bucketfs_host>:<bucketfs_port>
```

If inaccessible, fix BucketFS ACL/network before retrying virtual schema creation.

### 3. Validate source connection independently

Test source connectivity from Exasol path with a minimal JDBC import probe:

```sql
SELECT *
FROM (
  IMPORT FROM JDBC AT YOUR_CONNECTION_NAME
  STATEMENT 'SELECT * FROM EXT_SCHEMA_NAME.EXT_TABLE_NAME'
);
```

If this fails, investigate credentials, firewall, TLS, or source-side access.

### 4. Verify source/dialect support

Confirm target source is supported by selected virtual schema adapter and dialect.

## Result

`CREATE VIRTUAL SCHEMA` succeeds once driver deployment, BucketFS access, and source connectivity are all valid.

## References

- <https://docs.exasol.com/database_concepts/virtual_schemas.htm>
- <https://docs.exasol.com/db/latest/administration/on-premise/manage_drivers/add_jdbc_driver.htm>
- <https://github.com/EXASOL/virtual-schemas>

---

_We welcome feedback on this troubleshooting article._
