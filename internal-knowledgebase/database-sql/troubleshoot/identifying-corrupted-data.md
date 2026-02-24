---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: database-sql
title: "Identifying corrupted data in Exasol"
summary: "Trace checksum/blocksize backup errors from block ID to affected table and schema."
---
# Identifying corrupted data in Exasol

## Problem

Backup or read operations fail with block-level corruption errors (for example checksum mismatch or blocksize mismatch).

These errors usually include a corrupted block ID that can be traced to the owning object.

## Example indicator

```text
ERROR blocksize mismatch at reading of block 621194899456 ...
ERROR Checksum mismatch during backup occurred
```

Corrupted block ID in this example: `621194899456`.

## Step 1: Generate object-table dump

To map block IDs to objects, trigger object table dump in PDD/ObjectServer logs.

```shell
confd_client db_stop db_name: dev
confd_client db_configure db_name: dev params_add: '[-objectTableDump=2]'
confd_client db_start db_name: dev
```

After logs are generated, remove the parameter immediately:

```shell
confd_client db_stop db_name: dev
confd_client db_configure db_name: dev params_delete: '[-objectTableDump=2]'
confd_client db_start db_name: dev
```

## Step 2: Find object containing the corrupted block

Search logs for block ID:

```shell
grep -i 621194899456 /exa/logs/db/dev/*
```

Capture fields such as:

- Object `Name`
- `AuthorisationBase`
- `datablocks` list containing the block ID

## Step 3: Trace parent chain to table and schema

Use the found `AuthorisationBase` iteratively to resolve parent objects.

1. Search first-level parent:

```shell
grep -i <authorisation_base_id> /exa/logs/db/dev/*
```

1. Continue tracing until object types resolve to:

- `DBTableBasis` (table)
- `SchemaSchemaBasis` (schema)

Useful patterns:

```shell
grep -B 10 -i <authorisation_base_id> /exa/logs/db/dev/*
grep -B 16 -i <authorisation_base_id> /exa/logs/db/dev/*
```

## Step 4: Decide remediation

After identifying affected table and schema, choose recovery action.

Preferred order:

1. Attempt data salvage/export where possible.
1. Evaluate restore/repair options.
1. Drop and rebuild affected object only if recovery is not feasible.

Example destructive action:

```sql
DROP TABLE DEMO_SCHEMA_NAME.CORRUPTED_TABLE;
```

## Result

The incident is resolved when corrupted object is identified and a controlled remediation (recover, rebuild, or replace) is completed.

---

_We welcome feedback on this troubleshooting article._
