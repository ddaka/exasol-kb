---
tool_name: confd_client
doc_type: guide
category: system
title: "SaaS - How to find Customer Database UUID and Cluster UUID based on provided Account UUID"
summary: "Resolve database and cluster UUIDs from an account UUID using SaaS confd commands."
---
# SaaS - How to find Customer Database UUID and Cluster UUID based on provided Account UUID

## Purpose

Find `db_uuid` and `cluster_uuid` for operational workflows (support, logs, lifecycle commands).

Related runbooks:
- [`how-to-get-log-files-from-exasol-saas-systems-temporary.md`](../troubleshoot/how-to-get-log-files-from-exasol-saas-systems-temporary.md)
- [`how-to-use-confd-client-command-to-interact-with-saas.md`](how-to-use-confd-client-command-to-interact-with-saas.md)

## Prerequisites

- Access to SaaS management node.
- `confd_client` permissions for SaaS listing commands.
- Instance access guidance: [`saas-how-to-connect-to-the-instance.md`](saas-how-to-connect-to-the-instance.md)

## Step 1: get database UUIDs from account UUID

Example:

```bash
confd_client -c saas_db_list -a '{"account_uuid":"tkkMsfkEQYyisQhs0tRArg"}' -j
```

Optional extraction:

```bash
confd_client -c saas_db_list -a '{"account_uuid":"tkkMsfkEQYyisQhs0tRArg"}' -j | jq -r '.[].uuid'
```

## Step 2: get cluster UUIDs from database UUID

Example using one selected DB UUID:

```bash
confd_client -c saas_cluster_list -a '{"account_uuid":"tkkMsfkEQYyisQhs0tRArg", "db_uuid":"_va98QI2S72Ok2OiOrY5nA"}' -j
```

Optional extraction:

```bash
confd_client -c saas_cluster_list -a '{"account_uuid":"tkkMsfkEQYyisQhs0tRArg", "db_uuid":"_va98QI2S72Ok2OiOrY5nA"}' -j | jq -r '.[].uuid'
```

## Notes

- `-j` is optional but recommended for machine-readable output.
- Keep both `account_uuid` and `db_uuid` in ticket notes to speed up follow-up operations.
