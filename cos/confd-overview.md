---
tool_name: confd_client
doc_type: guide
category: General Overview
---

# confd_client Overview

## What is ConfD?

ConfD (Configuration Daemon) is a low-level API for performing all administrative actions in Exasol. It is available on all nodes in a deployment and provides a unified interface for cluster and database management.

You can interact with ConfD in two ways:
1. **Directly** — Using the command-line tool `confd_client`
2. **Indirectly** — Through XML-RPC in your own Python programs

## How ConfD Works

ConfD abstracts administration tasks into **jobs** with parameters. Jobs are executed asynchronously through a pipeline:

| Stage | Description |
|-------|-------------|
| **submitted** | Job ID received, waiting in queue |
| **scheduled** | Scheduler accepted the job, started execution |
| **executed** | Commands are being executed |
| **finished** | All commands executed and stored (final stage for read-only jobs) |
| **committed** | All nodes received changes, configuration synchronized (final stage for write jobs) |

## Using confd_client

### Accessing confd_client

Connect to the Cluster Operating System (COS) namespace:

```bash
c4 connect -t <DEPLOYMENT>[.<NODE>]/cos
```

Then run any confd_client command. When using `confd_client` on COS, jobs execute with the logged-in user (usually `root`). No extra authentication step is required.

### Command Syntax

**Inline YAML** (most common):

```bash
confd_client db_start db_name: Exasol
```

**JSON arguments**:

```bash
confd_client -c db_start -a '{db_name: Exasol}'
```

### Common Examples

```bash
confd_client db_start db_name: MyDatabase
confd_client db_stop db_name: MyDatabase
confd_client db_state db_name: MyDatabase
confd_client db_list
confd_client node_list
confd_client st_volume_list
```

## Using XML-RPC in Python

For programmatic access, connect via XML-RPC on port 20003:

```python
import xmlrpc.client, ssl

database_host = '203.0.113.42:20003'
user = 'admin'
pw = 'exasol'

connection_string = f'https://{user}:{pw}@{database_host}'
sslcontext = ssl._create_unverified_context()
conn = xmlrpc.client.ServerProxy(connection_string, context=sslcontext, allow_none=True)

conn.job_exec('db_start', {'params': {'db_name': 'Exasol'}})
```

## EXAConf Configuration File

Each node contains `/exa/etc/EXAConf` storing configuration settings managed by ConfD jobs.

**Important**: Manual editing is not recommended. Only edit when requested by Exasol Support. Use `general_settings` and `settings_list` jobs to manage EXAConf values.

## ConfD Job Categories

ConfD provides 150+ jobs organized into these categories:

- **Database**: `db_start`, `db_stop`, `db_create`, `db_configure`, `db_backup_*`, `db_snapshot_*`, etc.
- **Storage Volumes**: `st_volume_create`, `st_volume_delete`, `remote_volume_*`, `object_volume_*`, etc.
- **Storage Devices**: `st_device_add`, `st_device_remove`, `st_node_*`, etc.
- **Cluster Nodes**: `node_add`, `node_remove`, `node_suspend`, `node_resume`, etc.
- **BucketFS**: `bucket_add`, `bucket_delete`, `bucketfs_add`, `bucketfs_modify`, etc.
- **Users & Groups**: `user_create`, `user_delete`, `group_create`, `group_delete`, etc.
- **Licensing**: `license_upload`, `license_download`, `license_info`, `license_run_check`
- **Infrastructure**: `infra_db_scale`, `infra_instances_add`, `infra_firewall_update`, etc.
- **Plugins**: `plugin_add`, `plugin_remove`, `plugin_info`, `plugin_list`
- **Monitoring**: `monitoring_start`, `monitoring_stop`
- **System**: `general_settings`, `settings_list`, `cert_update`, `update_system`, `log_collect`

## Important: Database vs COS Services

There is NO `systemctl restart exasol` command. Exasol databases are managed through `confd_client`, NOT systemctl.

The only systemctl services are on the **host machine** (not COS):

```bash
systemctl --user status c4
systemctl --user status c4_cloud_command
```

These manage COS infrastructure, not databases. **Always use confd_client to manage databases.**

## Permissions

Jobs are restricted by OS users and groups:

- **root**: Full administrative access
- **exaadm**: General Exasol administration
- **exadbadm**: Database administration

## Best Practices

1. Always check job status after submission (jobs are asynchronous)
2. Use read-only jobs (`db_info`, `node_list`) to verify state before changes
3. Review job parameters carefully — many operations are destructive
4. Test in non-production environments first
5. Use `confd_client --help` for specific job syntax
