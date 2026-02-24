---
tool_name: internal-knowledgebase
doc_type: guide
category: database-sql
title: "Set Up a PostgreSQL FDW Connection to Exasol"
summary: "Build and configure jdbc_fdw with the Exasol JDBC driver to query Exasol tables from PostgreSQL via foreign tables."
---

# Set Up a PostgreSQL FDW Connection to Exasol

## Overview

This guide explains how to connect PostgreSQL to Exasol using `jdbc_fdw` and the Exasol JDBC driver.

Scope note:

- Procedure is based on Debian-style PostgreSQL Docker environments (for example `postgres:15-bookworm`).
- Package names and build steps may differ on other distributions.

## Prerequisites

- Running PostgreSQL instance (PostgreSQL 14+ recommended).
- Network access from PostgreSQL host/container to Exasol.
- Exasol JDBC driver (`exajdbc.jar`).

## Procedure

### 1. Install dependencies and build `jdbc_fdw`

```bash
apt-get update
apt-get install -y git make gcc wget unzip postgresql-server-dev-15 openjdk-17-jdk

java -version

git clone https://github.com/pgspider/jdbc_fdw.git /tmp/jdbc_fdw_pg15
cd /tmp/jdbc_fdw_pg15

wget https://ftp.postgresql.org/pub/source/v15.4/postgresql-15.4.tar.gz -O /tmp/postgresql-15.4.tar.gz
tar xzf /tmp/postgresql-15.4.tar.gz -C /tmp
cp /tmp/postgresql-15.4/contrib/contrib-global.mk /tmp/jdbc_fdw_pg15/

export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export LD_LIBRARY_PATH=$JAVA_HOME/lib/server

make USE_PGXS=1 LDFLAGS="-L$JAVA_HOME/lib/server"
make install USE_PGXS=1 LDFLAGS="-L$JAVA_HOME/lib/server"

echo "/usr/lib/jvm/java-17-openjdk-amd64/lib/server" > /etc/ld.so.conf.d/java.conf
ldconfig

mkdir -p /usr/share/java
cp /tmp/exajdbc.jar /usr/share/java/exajdbc.jar
```

### 2. Enable extension in PostgreSQL

```sql
CREATE EXTENSION jdbc_fdw;
```

### 3. Create FDW server and user mapping

```sql
CREATE SERVER exasol_server
FOREIGN DATA WRAPPER jdbc_fdw
OPTIONS (
  drivername 'com.exasol.jdbc.EXADriver',
  url 'jdbc:exa:<EXASOL_HOST>/<FINGERPRINT>:8563;schema=VIT',
  jarfile '/usr/share/java/exajdbc.jar'
);
```

```sql
CREATE USER MAPPING FOR postgres
SERVER exasol_server
OPTIONS (username 'EXA_USER', password 'EXA_PASSWORD');
```

### 4. Create foreign table and test query

```sql
CREATE FOREIGN TABLE exasol_vit_tt (
  qwe VARCHAR(100)
)
SERVER exasol_server
OPTIONS (schema_name 'VIT', table_name 'TT');
```

```sql
SELECT * FROM exasol_vit_tt;
```

## Validation

- `CREATE EXTENSION jdbc_fdw` succeeds.
- Foreign table can be created without driver/linker errors.
- `SELECT` from foreign table returns expected Exasol data.

## Notes

- Replace placeholders (`<EXASOL_HOST>`, `<FINGERPRINT>`, credentials, schema/table names) with environment values.
- If JVM linker errors occur, re-check `JAVA_HOME`, `LD_LIBRARY_PATH`, and `ldconfig` registration.
