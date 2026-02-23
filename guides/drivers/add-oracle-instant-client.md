---
tool_name: confd_client
doc_type: guide
category: driver-management
technical_entities:
  - Oracle Instant Client
  - BucketFS
  - IMPORT
  - ORA
summary: >
  How to add Oracle Instant Client to Exasol — compatible version matrix,
  BucketFS upload, and verification with IMPORT FROM ORA.
---

# Add Oracle Instant Client

Oracle Instant Client enables loading data using `IMPORT FROM ORA` syntax.

## Prerequisites

- Traffic allowed on BucketFS port (default: **2581**)
- Write password changed for the default bucket

## Compatible Versions

| Exasol Version           | Oracle Client Version       | Notes                              |
|--------------------------|-----------------------------|------------------------------------|
| 8.31.0 and earlier       | 12.1.0.2.0                  | Standard Oracle downloads page     |
| 8.32.0 to 2025.1.8       | 23.5.0.24.07                | Direct download links only         |
| 2025.2 and later         | 23.9.0.25.07                | Direct download links only         |

## Upload to BucketFS

Default Oracle client path: `/buckets/bfsdefault/default/drivers/oracle/`

```bash
curl -v --insecure -X PUT \
  -T instantclient-basic-linux.x64-23.5.0.24.07.zip \
  https://w:$WRITE_PW@$DATABASE_NODE_IP:2581/default/drivers/oracle/instantclient-basic-linux.x64-23.5.0.24.07.zip
```

## Verification

```sql
CREATE OR REPLACE CONNECTION OCI_ORACLE
  TO '192.168.99.103:1521/xe'
  USER 'system'
  IDENTIFIED BY 'oracle';

SELECT * FROM (
  IMPORT FROM ORA AT OCI_ORACLE
  STATEMENT 'select ''Connection works'' from dual'
);
```
