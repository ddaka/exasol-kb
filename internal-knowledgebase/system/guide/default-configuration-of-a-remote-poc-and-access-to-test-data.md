---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "Remote PoC Default Configuration and Test Data Access"
summary: "Operational guide for remote PoC network layout, staging host usage, and data access patterns via SFTP and SSH tunnel forwarding."
---

# Remote PoC Default Configuration and Test Data Access

## Overview

This guide describes the standard remote PoC setup and practical patterns for importing test data and connecting to external databases.

## Terminology

- `Remote PoC`: customer test cluster hosted in Exasol labs and accessed remotely.
- `On-site PoC`: customer-hosted cluster in customer data center.
- `MCTC`: mobile customer test cluster.
- `SCTC`: stationary customer test cluster.

## Default Remote PoC Network Pattern

Remote PoC access is typically provided via:

- VPN, or
- staging server (jump host)

Standard staging host mapping:

| Attribute | Value |
| --- | --- |
| External hostname | `ctcX.exasol.com` |
| Internal hostname | `ext.ctcX.exasol.com` |
| Internal staging IP | `10.60.X.2` (`ext`) |
| License server IP | `10.60.X.10` (`n10` / license) |
| DB node IP range | `10.60.X.11..15` (`n11..n15`) |

`X` value:

- SCTC: `X = SCTC number`
- MCTC: `X = MCTC number + 10`

Examples:

- `ctc1.exasol.com` -> SCTC1
- `ctc11.exasol.com` -> MCTC1

## Import CSV Data via Staging Host

### Notes

- FTP usually restricts access to user home directory.
- For paths like `/data`, prefer SFTP or mount `/data` into user home.
- Ensure DNS search domain setup matches remote PoC naming conventions.

### Connection template

```sql
SET DEFINE &;

DEFINE server   = ctc12;
DEFINE ip       = 10.60.12.2;
DEFINE user     = my_user;
DEFINE password = my_pass;
DEFINE type     = sftp;

CREATE OR REPLACE CONNECTION &type._&server
TO '&type.://&ip'
USER '&user'
IDENTIFIED BY '&password';

-- basic connection test
SELECT * FROM (
  IMPORT INTO (ls VARCHAR(2000000))
  FROM CSV AT &type._&server
  FILE '/home/&user/'
);
```

## Import from External Databases

### Scenario A: Staging host can reach target DB port

On staging host `ctcX`, open local forward (example PostgreSQL):

```bash
screen -S postgresql_forwarding
ssh my_user@localhost -L 10.60.X.2:5432:public.postgresql.database.com:5432
# detach: Ctrl+a then d
```

Create Exasol connection:

```sql
CREATE CONNECTION my_postgresql
TO 'jdbc:postgresql://10.60.X.2:5432/db_name?ssl=true&sslfactory=org.postgresql.ssl.NonValidatingFactory'
USER 'db_user'
IDENTIFIED BY 'db_password';
```

### Scenario B: Staging host cannot reach target DB port directly

Requirements:

- customer-side jump host with access to target DB
- `GatewayPorts` enabled on Exasol jump host (`GatewayPorts clientspecified`)

Validation:

```bash
grep GatewayPorts /etc/ssh/sshd_config
```

Remote forward from customer jump host:

```bash
ssh my_user@ctcX.exasol.com -R 10.60.X.2:5432:private.postgresql.database.com:5432
```

Then create the same Exasol JDBC connection as Scenario A.

## Validation Checklist

- DNS/NTP checks pass on cluster and staging host.
- Tunnel endpoint is reachable before creating JDBC connection.
- Exasol `CREATE CONNECTION` and test query/import succeed.
