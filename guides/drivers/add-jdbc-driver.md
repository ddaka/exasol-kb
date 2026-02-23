---
tool_name: confd_client
doc_type: guide
category: driver-management
technical_entities:
  - JDBC driver
  - BucketFS
  - IMPORT
  - EXPORT
  - settings.cfg
summary: >
  How to add a third-party JDBC driver to Exasol — create settings.cfg,
  upload driver jar and config to BucketFS, and verify with IMPORT.
---

# Add JDBC Driver

Exasol requires JDBC drivers to load data from third-party sources using
`IMPORT`. Drivers are uploaded and managed through BucketFS. The Exasol JDBC
driver comes pre-installed.

## Prerequisites

- JDBC driver jar file(s) downloaded locally
- Traffic allowed on BucketFS port (default: **2581**)
- Write password changed for the default bucket

## Step 1: Create a Configuration File

Create `settings.cfg`:

```
DRIVERNAME=$MY_DRIVERNAME
PREFIX=$PREFIX
FETCHSIZE=100000
INSERTSIZE=-1
```

- `DRIVERNAME` — unique name used in `IMPORT`/`EXPORT` `driver` clause
- `PREFIX` — JDBC URL prefix (from driver vendor docs)
- Do not modify `FETCHSIZE`/`INSERTSIZE` unless instructed by Support

### Optional Parameters

| Parameter    | Description                                               |
|--------------|-----------------------------------------------------------|
| `JAR=`       | Force only specific jar files to be loaded                |
| `DRIVERMAIN=`| Specify main JDBC driver class explicitly                 |
| `NOSECURITY=YES` | Disable security manager (for permission issues)     |

Every line including the final line must end with a newline character (LF).

**Example** (Exasol JDBC driver):

```
DRIVERNAME=EXASOL_JDBC
PREFIX=jdbc:exa:
FETCHSIZE=100000
INSERTSIZE=-1
```

## Step 2: Upload to BucketFS

Default JDBC driver path: `/buckets/bfsdefault/default/drivers/jdbc/`

```bash
curl -v --insecure -X PUT -T settings.cfg \
  https://w:$WRITE_PW@$DATABASE_NODE_IP:2581/default/drivers/jdbc/exasol/settings.cfg

curl -v --insecure -X PUT -T exajdbc.jar \
  https://w:$WRITE_PW@$DATABASE_NODE_IP:2581/default/drivers/jdbc/exasol/exajdbc.jar
```

> The `--insecure` flag bypasses TLS certificate verification. Only use when
> certificate verification is not possible and you trust the server.

## Verification

```sql
CREATE OR REPLACE CONNECTION EXASOL_CONNECTION_JDBC
  TO 'jdbc:exa:192.168.0.72:8563'
  USER 'SYS'
  IDENTIFIED BY 'exasol';

SELECT * FROM (
  IMPORT FROM JDBC AT EXASOL_CONNECTION_JDBC
  STATEMENT 'select ''Connection works'' '
);
```
