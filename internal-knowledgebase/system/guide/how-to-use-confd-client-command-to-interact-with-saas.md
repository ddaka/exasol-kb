---
tool_name: confd_client
doc_type: guide
category: system
title: "How to use confd_client to interact with SaaS"
summary: "Run SaaS-specific ConfD jobs from SaaS management namespace to list, inspect, and control SaaS accounts, databases, and clusters."
---
# How to use confd_client to interact with SaaS

## Purpose

Use `confd_client` to execute SaaS control-plane jobs (`saas_*`) from the SaaS management environment.

## Access options

- Shell execution on SaaS management instance (covered here).
- REST API execution (out of scope for this guide).

## Prerequisites

- Access to SaaS management EC2 instance.
- Access to correct namespace (COS/Linux context where SaaSDB control jobs are available).
- `account_uuid`, `db_uuid`, `cluster_uuid` as needed.

If needed, connect to namespace using environment-specific SSH command (example):

```shell
sudo ssh -i /home/ubuntu/.ccc/play/local/*/main/11/root/root/.ssh/id_rsa localhost -p 20002
```

## 1) Discover available SaaS jobs

```shell
confd_client -c list | grep saas
```

## 2) List accounts

```shell
confd_client -c saas_accounts_list
confd_client -c saas_accounts_list -j
```

## 3) List DBs for account

```shell
confd_client -c saas_db_list -a '{"account_uuid":"<ACCOUNT_UUID>"}' -j
```

## 4) List clusters for DB

```shell
confd_client -c saas_cluster_list -a '{"account_uuid":"<ACCOUNT_UUID>","db_uuid":"<DB_UUID>"}' -j
```

## 5) DB lifecycle actions

Start DB:

```shell
confd_client -c saas_db_start -a '{"account_uuid":"<ACCOUNT_UUID>","db_uuid":"<DB_UUID>"}' -j
```

Stop DB:

```shell
confd_client -c saas_db_stop -a '{"account_uuid":"<ACCOUNT_UUID>","db_uuid":"<DB_UUID>"}' -j
```

Delete DB:

```shell
confd_client -c saas_db_delete -a '{"account_uuid":"<ACCOUNT_UUID>","db_uuid":"<DB_UUID>"}' -j
```

Get DB details:

```shell
confd_client -c saas_db_info -a '{"account_uuid":"<ACCOUNT_UUID>","db_uuid":"<DB_UUID>"}' -j
```

## Notes

- Default output is YAML-like; add `-j` for JSON.
- Run commands only in authorized environment/profile.

## De-duplication note

Canonical `confd_client` usage and syntax reference:

- `documents/cos/confd-overview.md`


