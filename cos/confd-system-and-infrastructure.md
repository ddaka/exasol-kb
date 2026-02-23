---
tool_name: confd_client
doc_type: reference
category: System and Infrastructure
subcommands: general_settings, settings_list, cert_update, license_download, license_info, license_run_check, license_upload, log_collect, debug_collect, monitoring_start, monitoring_stop, update_system, infra_db_scale, infra_db_start, infra_db_stop, infra_deployment_metadata_get, infra_firewall_update, infra_instances_add, infra_instances_remove, infra_instances_start, infra_instances_stop, infra_instances_update, infra_overview_get, infra_resources_update, infra_worker_db_add, infra_worker_db_remove, conn_get_remote_nid, plugin_add, plugin_info, plugin_list, plugin_remove, affinity_reset
---

# confd_client — System, Licensing, and Infrastructure

## Overview

Commands for system administration, licensing, monitoring, updates, cloud infrastructure management, plugins, and general settings.

All commands run inside the COS namespace (SSH port 20002).

## general_settings

This job changes EXAConf file settings that do not need a parent job.

**Permissions**: Users: root | Groups: root, exaadm

**Parameters**:

- `changes` (dict, required): 'Change all settings in the section name or subname. For example, {"Global": {"NameServers": "4.4.4.4", "XMLRPCPort": 20003} changes the NameServers and XMLRPCPort settings to the new settings.'

**Examples**:

```bash
confd_client general_settings {changes: {Global: {NameServers: 4.4.4.4, XMLRPCPort: 20003}}}
```

## settings_list

The job lists current settings in the EXAConf file, a specific section in
the file, or a specific subsection.

**Permissions**: Users: root | Groups: root, exaadm

**Parameters**:

- `sections` (dict, optional): 'All settings within a specified section or subsection name. For example, the following searches the Global section for the settings in the NameServers and XMLRPCPort subsections: {"Global": ["NameSer

**Examples**:

```bash
confd_client settings_list {sections: {Global: [NameServers, XMLRPCPort]}}
```

## cert_update

This job uploads a new TLS certificate which is used for all databases,
buckets and configuration APIs.

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `cert` (str, required): The entire TLS certificate chain as a string.
- `key` (str, required): The entire TLS private key as a string.
- `ca` (str, optional): The entire TLS certificate authority as a string.

## license_download

This job returns the contents of the license file as a string.

**Permissions**: Users: root | Groups: root, exaadm

## license_info

This job returns the fields of the license.

**Permissions**: Users: root | Groups: root, exaadm

## license_run_check

This job checks the running databases against the license limits.

**Permissions**: Users: root | Groups: root, exaadm

## license_upload

This job uploads a new license to the cluster. It may take a few minutes for
the new license to take effect.

**Permissions**: Users: root | Groups: root, exaadm

**Parameters**:

- `license` (str, required): The entire contents of the license file as a string.

**Examples**:

```bash
confd_client license_upload {license: '...'}
```

## log_collect

This job retuns log events that fit the search paramater values.

      13:13:00', tag: TagWord}

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `services` (list, required): Finds log files for specified services in a comma separated list. To get a list of available services, run 'logd_client --show-services'.
- `dbs` (list, optional): Finds log events from specified database IDs in a comma seperated list.
- `default_interval` (str, optional): 'Finds log events with specified intervals in the following format: <num>w <num>d <num>h <num>m <num>s or <num> in seconds. By default, the interval is 10 minutes.'
- `error_level` (str, optional): 'Finds log events with the specified error priorities in a comma separated list. The following lists valid values: ERR, WARN, NOTICE, and INFO.'
- `nodes` (list, optional): Finds log events from node IDs in a comma seperated list.
- `start_time` (str, optional): 'Sets the time and date to start collecting log data. Use the following format: %F %T. For example: 2019-01-04 13:13:00.'
- `stop_time` (str, optional): 'Sets the time and date to stop collecting log data. Use the following format: %F %T. For example: 2019-01-04 13:15:00.'
- `tag` (str, optional): A string used to identify the current stream of log events. The first time you call the job with a custom string the tag is created with the given name. The next time you run this command and use the 

**Examples**:

```bash
confd_client log_collect {end_time: '2019-01-04 13:15:00', services: [ConfD, DWAd, HealthD], start_time: '2019-01-04
```

## debug_collect

This job retuns log events about debug information that fits the search
parameter values.

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `backtraces` (list, optional): 'A comma seperated list of backtraces to be collected. The list can include: EXASolution server processes, EXASolution SQL processes, EXAClusterOS processes, or ETL JDBC Jobs.'
- `debug_info` (list, optional): 'Any combination of of following: EXAClusterOS logs, EXAStorage metadata, or Coredumps.'
- `end_date` (str, optional): 'Ends the log at the exact date using the following format: yyyy-mm-dd.'
- `exasolution_log_type` (list, optional): 'Any combination of the following: SQL processes, Server processes, or Session <id>. If no log types are listed, all will be included.'
- `exasolution_logs` (list, optional): All databases or any specific database name listed. If no databases are listed, all will be included.
- `no_open_files` (bool, optional): 'If true, includes only archived files. Valid values are: True or False.'
- `nodes` (list, optional): A comma seperated list of node IDs in this cluster to include in the log. If no node IDs are listed, the log will include all nodes in that current cluster.
- `only_open_files` (bool, optional): 'If true, includes only active files. Valid values are: True or False.'
- `outfile` (str, optional): Location to store debug information. For example, remote:VolumeName,RemoteVolumePath.
- `start_date` (str, optional): 'Starts the log at the exact date using the following format: yyyy-mm-dd.'
- `timeout` (int, optional): Sets maximal duration in seconds for collecting debug information. The default is 1800 seconds (10 minutes).

**Examples**:

```bash
confd_client debug_collect {debug_info: Coredumps}
```

## monitoring_start

Will add a new EXAConf section 'Monitoring' with all the important
configuration parameters.

    sasl_user: name}

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `broker` (str, required): Kafka broker that monitoring service will be connecting to.
- `cluster_id` (str, required): ID To identify current cluster.
- `sasl_password` (str, required): Password for SASL authentication with a Kafka broker.
- `sasl_user` (str, required): Username for SASL authentication with a Kafka broker.

**Examples**:

```bash
confd_client monitoring_start {broker: 'test.exasol.com:9090', cluster_id: 98f267a3, sasl_password: password,
```

## monitoring_stop

Will remove EXAConf section 'Monitoring' and stop monitoring service.

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

## update_system

Handle update operations for the COS cluster.

**Permissions**: Groups: root

**Parameters**:

- `tags` (bool, optional): If set to true, updates all tags with packages from the artifacts repository service. Valid values are true and false.
- `target` (str, optional): Specifies the target to update. By default, the update stream specified by c4 is used as the target.

**Examples**:

```bash
confd_client update_system {tags: true}
```

## infra_db_scale

This job updates a node(s) of the cluster and updates the database
configuration.

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `db_name` (str, required): The name of the database. The database will be stopped.
- `instance_type` (str, required): 'Enter the instance type that fits your database needs. Valid values include: 5d.large, r5d.xlarge, and r5d.2xlarge.'
- `offload_enabled` (bool, optional): Whether or not offload is enabled for this worker db.
- `offload_force` (bool, optional): All sessions which cannot be moved will have their currently running statements killed.
- `offload_timeout` (int, optional): The client/statement will wait up to timeout in seconds for the sessions to be moved.

**Examples**:

```bash
confd_client infra_db_scale {db_name: DB1, instance_type: r5d.xlarge}
```

## infra_db_start

Start an existing database and all of its nodes.

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `db_name` (str, required): The name of the database you want to start.

**Examples**:

```bash
confd_client infra_db_start {db_name: DB1}
```

## infra_db_stop

Stops an existing database in the cluster and shutdown the instances.

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `db_name` (str, required): The name of the database you want to stop.
- `logtimeout` (int, optional): The time limit for rotating logs on all nodes that are being stopped. The default is 180 seconds (3min).
- `offload_enabled` (bool, optional): Whether or not offload is enabled for this worker db.
- `offload_force` (bool, optional): All sessions which cannot be moved will have their currently running statements killed.
- `offload_timeout` (int, optional): The client/statement will wait up to timeout in seconds for the sessions to be moved.

**Examples**:

```bash
confd_client infra_db_stop {db_name: DB1}
```

## infra_deployment_metadata_get

This job provides deployment metadata for the cluster.

  {}

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

## infra_firewall_update

This job updates the firewall rules of the deployment.

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `egress_rules` (list, optional): The list with key/value pair dictionaries where each dictionary is an egress firewall rule.
- `ingress_rules` (list, optional): The list with key/value pair dictionaries where each dictionary is an ingress firewall rule.

**Examples**:

```bash
confd_client infra_firewall_update {ingress_rules: [{CidrIp: 0.0.0.0/0, FromPort: 22, IpProtocol: tcp, ToPort: 22}]}
```

## infra_instances_add

This job adds an additional node(s) to a cluster. You can create the new
node(s) using an existing node configuration or the cluster configuration.

    num_nodes: 1}

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `nid` (int, required): The ID of a node with the configuration that should be used to create the new node(s).
- `num_nodes` (int, required): The number of nodes you want to create.
- `cloud_zone` (str, optional): The encoded geographical region that stores the cloud resources. For example, Europe (Frankfurt) is eu-central-1.
- `disks` (list, optional): The storage disk(s) to which you want to add the new node(s).
- `is_workernode` (bool, optional): If the node is stored in a cluster other than the MAIN cluster, enter "True". Otherwise, enter "False".
- `metadata` (dict, optional): The key/value pair (key1:value1) used to identify the cluster specific metadata hash table to use for additional node configuration.
- `node_platform_config` (dict, optional): The cluster specific node configuration.
- `timeout` (int, optional): The time limit in seconds. The default is 1800 seconds (30min).

**Examples**:

```bash
confd_client infra_instances_add {cloud_zone: eu-central-1, disks: TestStorageDisk, is_workernode: true, nid: 11,
```

## infra_instances_remove

This job removes a node(s) from the cluster.

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `nids` (list, required): The ID of the node you want to remove.
- `metadata` (dict, optional): The key/value pair (key1:value1) used to identify the cluster specific metadata hash table to use for additional node configuration.
- `timeout` (int, optional): The time limit in seconds for the removal of the node(s). The default is 1800 seconds (30min).

**Examples**:

```bash
confd_client infra_instances_remove {nids: [11, 12], timeout: 2000}
```

## infra_instances_start

This job starts an existing node(s) of the cluster.

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `nids` (list, required): The ID(s) of a node(s) you want to start.

**Examples**:

```bash
confd_client infra_instances_start {nids: [11, 12]}
```

## infra_instances_stop

This job stops an existing node(s) of the cluster.

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `nids` (list, required): The ID(s) of the node(s) you want to stop.
- `timeout` (int, optional): The time limit in seconds to stop the node(s). The default is 1800 seconds (30min).

**Examples**:

```bash
confd_client infra_instances_stop {nids: [11, 12], timeout: 2000}
```

## infra_instances_update

This job updates nodes based on the cluster configuration.

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `nids` (list, required): The ID(s) of the node(s) you want to update with the given configuration.
- `node_platform_config` (dict, required): The cluster specific node configuration.
- `metadata` (dict, optional): The key/value pair (key1:value1) used to identify the platform specific metadata hash table to use for additional node configuration.

**Examples**:

```bash
confd_client infra_instances_update {metadata: 'Option1:Value1', nids: [11], node_platfrom_config: ClusterName}
```

## infra_overview_get

This job provides and overview of the cluster state (e.g., running,
stopped).

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

## infra_resources_update

This job updates all other resources of the respective platform.

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `node_platform_config` (dict, required): The cluster specific node configuration.
- `metadata` (dict, optional): The key/value pair (Option1:Value1) used to identify the platform specific metadata hash table to use for additional node configuration.

**Examples**:

```bash
confd_client infra_resources_update {node_platform_config: ClusterConfig}
```

## infra_worker_db_add

This job creates a worker cluster and adds necessary additional nodes.

    new_worker_db_name: new_worker_cluster_name}

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `instance_type` (str, required): 'Enter the instance type that fits your database needs. Valid values include: 5d.large, r5d.xlarge, and r5d.2xlarge.'
- `master_db_name` (str, required): The name of the main cluster.
- `new_worker_db_name` (str, required): The name for the new worker cluster.
- `cloud_zone` (str, optional): The encoded geographical region that stores the cloud resources. For example, Europe (Frankfurt) is eu-central-1.
- `db_params` (str, optional): A space-seperated list of extra database parameters which replaces the current settings. When used with other params jobs, such as params_add and/or params_delete, params is evaluated first. In genera
- `metadata` (dict, optional): The key/value pair (key1:value1) used to identify the cluster specific metadata hash table to use for additional node configuration.
- `port` (int, optional): Port that the database is reachable on. By default, the port is 8563.

**Examples**:

```bash
confd_client infra_worker_db_add {cloud zone: eu-central-1, instance_type: 5d.large, master_db_name: main_cluster_name,
```

## infra_worker_db_remove

This job deletes a worker cluster and removes its nodes.

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `worker_db_name` (str, required): The name of the worker cluster you want to remove.
- `offload_enabled` (bool, optional): Whether or not offload is enabled for this worker db.
- `offload_force` (bool, optional): All sessions which cannot be moved will have their currently running statements killed.
- `offload_timeout` (int, optional): The client/statement will wait up to timeout in seconds for the sessions to be moved.

**Examples**:

```bash
confd_client infra_worker_db_remove {worker_db_name: worker_cluster_name}
```

## conn_get_remote_nid

Return the physical node ID This job determines the physical node ID (root
node ID) of the calling process.

**Permissions**: Users: root | Groups: root, exastoradm, exaadm

## plugin_add

This job adds a new plugin to a bucket.

**Permissions**: Users: root | Groups: root, exaadm

**Parameters**:

- `plugin_name` (str, required): The name of the plugin.
- `bucket_name` (str, optional): The name of the bucket that will store the plugin. If the bucket does not already exist, a new one will be created.
- `bucketfs_name` (str, optional): The name of the BucketFS service where the bucket is stored.
- `dir` (str, optional): The folder name and location where the plugin will be stored. The default folder name is plugins. If you created your own bucket, you must create your own folder in which to store the plugin.

**Examples**:

```bash
confd_client plugin_add {bucket_name: Bucket1, bucketfs_name: BucketFS1, plugin_name: Plugin1}
```

## plugin_info

This job provides information about a plugin.

**Permissions**: Users: root | Groups: root, exaadm

**Parameters**:

- `plugin_name` (str, required): The name of the plugin.

**Examples**:

```bash
confd_client plugin_info {plugin_name: Plugin1}
```

## plugin_list

This job lists all plugins.

**Permissions**: Users: root | Groups: root, exaadm

## plugin_remove

This job removes a plugin.

**Permissions**: Users: root | Groups: root, exaadm

**Parameters**:

- `plugin_name` (str, required): Name of the plugin.

**Examples**:

```bash
confd_client plugin_remove {plugin_name: Plugin1}
```

## affinity_reset

This job resets affinities of nodes after the exainit script has run, and
selects a new master node if necessary.

**Permissions**: Groups: root, exaadm
