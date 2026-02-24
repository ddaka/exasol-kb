---
tool_name: c4
doc_type: guide
category: system
title: "Update customer databases on SaaS"
summary: "Manual SaaS customer database update procedure using c4 when automatic update does not apply."
---
# Update customer databases on SaaS

## Purpose

Most SaaS customer databases update automatically. Use this procedure for controlled manual updates when required.

## Prerequisites

- Access to SaaS production account and database access node.
- Target production tag (for example `@saas-prod-n`).
- Verified backups before update.
- Downtime window in place.

## Step 1: Schedule downtime

Create maintenance downtime before update:

- `create-downtime-schedule-for-saas.md`

## Step 2: Connect to instances

Connect to required nodes (access node and DB nodes) per SaaS access playbook.

## Step 3: Verify `c4` version

```shell
CCC_AWS_REGION=eu-west-1 c4 version
```

`c4` versions `@0.4.2-dev.1` and later support update execution from access node only.

If older, upgrade `c4` tooling first per internal process.

## Step 4: Execute update from access node

```shell
c4 update -P -t @saas-prod-n
```

Successful output should confirm update and cleanup completion.

## Step 5: Validate DB runtime state

If needed, enforce version and restart with `confd_client`:

```shell
confd_client -c infra_db_stop -a 'db_name: <db_name>'
confd_client -c db_configure -a '{db_name: <db_name>, version: <new_version>}'
confd_client -c infra_db_start -a 'db_name: <db_name>'
```

## Step 6: Failure handling

If update fails:

1. Confirm old version is still running.
1. Collect logs and update artifacts.
1. Escalate to R&D with evidence.

## Legacy note (older deployments)

Some older SaaS deployments may require additional snapshot-volume operations during update.

Warning: reformat operations can delete backups. Do not execute without explicit backup validation and approval.

## Reference

- c4 update docs: <https://github.com/exasol/ccc/blob/master/doc/update.md>


