---
tool_name: confd_client
doc_type: reference
category: Database Management
subcommands: db_start, db_stop, db_state, db_info, db_list, db_create, db_delete, db_configure, db_enlarge, db_reset, db_resume_nodes, db_suspend_nodes, db_add_reserve_nodes, db_remove_reserve_nodes, db_configure_kerberos
---

# confd_client — Database Management

## Overview

Commands for managing Exasol databases: creating, starting, stopping, configuring, deleting, and scaling database instances.

All commands run inside the COS namespace (SSH port 20002).

## db_start

Starts the database. The database must already exist and must not be
running.

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `db_name` (str, required): Name of an existing database to be started.
- `cluster` (str, optional): Which database clusters should be started. Can be a cluster name, '+all' or 'MAIN' (database name can be passed instead of 'MAIN').
- `maintenance` (bool, optional): If set to True, the database is started in maintenance mode. This should only be done on recommendation from support.
- `timeout` (int, optional): Number of seconds to wait for the database to be running. The default value is 0, which disables the timeout. If the timeout is reached, the operation is aborted.

**Examples**:

```bash
confd_client db_start {db_name: DB1}
confd_client db_start {db_name: DB1, timeout: 30}
```

## db_stop

Stops a running database. The database nodes will remain online and
available.

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `db_name` (str, required): Name of an existing database to be stopped.
- `cluster` (str, optional): Which database clusters should be stopped. Can be a cluster name, '+all' or 'MAIN' (database name can be passed instead of 'MAIN').
- `force` (bool, optional): When set to True, the database is forced to shutdown. This should only be used if the database cannot shut down normally.

**Examples**:

```bash
confd_client db_stop {db_name: DB1}
confd_client db_stop {db_name: DB1, force: true}
```

## db_state

This job returns the current state of the given database. Some common states
are 'Running' (meaning the database is online), 'Setup' (meaning the
database is offline), 'Startup' (meaning the database is starting up), and
'Shutdown' (meaning the database is shutting down).

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `db_name` (str, required): Name of an existing database.

**Examples**:

```bash
confd_client db_state {db_name: exa_db}
```

## db_info

Returns a dictionary with the main database and volume parameters.

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `db_name` (str, required): Name of an existing database.

**Examples**:

```bash
confd_client db_info {db_name: exa_db}
```

## db_list

This job returns the name of all created databases.

**Permissions**: Users: root | Groups: root, exaadm, exadbadm, exausers

## db_create

Creates a new database. A database with this name or UUID must not already
exist.

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `data_volume_name` (str, required): Volume name for the database data. The volume must be created beforehand. This parameter may be substituted by data_volume_id.
- `db_name` (str, required): Unique name of the new database.
- `mem_size` (str, required): Amount of memory (RAM) allocated to the database in MiB, GiB, or TiB, e.g. "2048 MiB".
- `nodes` (list, required): List of active and reserved node IDs for this database. The node IDs are integers.
- `num_active_nodes` (int, required): Number of active nodes in the database.
- `version` (str, required): Version of the database, e.g. "8.0.0".
- `additional_sys_passwd_hashes` (str, optional): A semi-colon seperated list of password hashes that allow authentication as SYS independent of the SYS password set.
- `auto_start` (bool, optional): Specifies if the database starts on cluster boot automatically. The default value is True.
- `cache_volume_disk` (str, optional): Specify disks to use for cache in the format <diskname>:<volumesize>, e.g. "disk1:7 GiB".
- `cloud_data_volume_name` (str, optional): The name of the cloud-based storage volume to use (e.g. "DataVolume3"). The volume must be created beforehand.
- `data_volume_id` (int, optional): Volume ID for the database data. data_volume_id can substitute data_volume_name. If both data_volume_id and data_volume_name are specified, data_volume_id is used instead.
- `db_uuid` (str, optional): UUID for the database cluster in base64 format. Name is used as fallback if it is a valid UUID.
- `default_sys_passwd_hash` (str, optional): The hash used to set the default SYS password during database creation.
- `enable_auditing` (bool, optional): Enables or disables auditing for the database. The default value is True.
- `initial_sql` (str, optional): Absolute path to a YAML file with SQL statements to run during the first database startup.
- `interfaces` (str, optional): Comma-seperated list of network interfaces to use for the database. Leave empty to use all possible network interfaces. The default value is "".
- `jdbc_urls` (list|tuple, optional): 'List of URLs used to search for JDBC drivers, like: bucketfs://bfs2/bucket3/jdbcdir or file:///exa/etc/jdbc_drivers.'
- `ldap_server` (str, optional): Comma-separated list of LDAP Servers to use for remote database authentication, e.g. "ldap[s]://192.168.16.10". Each server must start with "ldap://" or "ldaps://".
- `master_database` (str, optional): The name of the master database if a worker database is being created.
- `oracle_url` (str, optional): 'An URL used to search for oracle instant client, like: bucketfs://bfs2/bucket3/oracle_dir.'
- `owner` (list|tuple, optional): Owner of the new database, either (user id, group id) or (user name, group name). Defaults to ("exadefusr", "exausers").
- `params` (str, optional): A space-seperated list of extra database parameters that are set when the database is created. In general, parameters are in the format "-param=value", e.g. "-param1=1 -param2=0".
- `port` (int, optional): Port that the database is reachable on. If no port is specified, the database is created with port 8563.
- `snapshot_sync_volume` (str, optional): The name of the cloud-based snapshot sync volume to use (e.g. "SnapshotSyncVolume1"). The volume must be created beforehand.
- `volume_move_delay` (int, optional): Specifies the delay in seconds after which the volume on a failed node is moved to the reserve node automatically. When no value is set, there is no delay.
- `volume_quota` (int, optional): Maximum size of the data volume in GiB. If the quota is reached, the database tries to shrink the volume on start-up if required and possible.

**Examples**:

```bash
confd_client db_create {data_volume_name: DataVolume1, db_name: exa_db, mem_size: 2048 MiB, nodes: [
confd_client db_create {data_volume_name: DataVolume2, db_name: exa_db2, enable_auditing: false, initial_sql: /tmp/initial_sql.yaml,
```

## db_delete

This job deletes a database. The database being deleted must already exist.

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `db_name` (str, required): Name of the existing database to be deleted.

**Examples**:

```bash
confd_client db_delete {db_name: DB1}
```

## db_configure

This job changes the configuration of a database. The database must be
offline to change its configuration.

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `db_name` (str, required): Name of an existing database.
- `additional_sys_passwd_hashes` (str, optional): A semi-colon seperated list of password hashes that allow authentication as SYS independent of the SYS password set.
- `auto_start` (bool, optional): Specifies if the database starts on cluster boot automatically.
- `cache_volume_disk` (str, optional): 'Specifies disks to use for cache: <diskname>:<volumesize>, e.g. "disk1:7 GiB".'
- `cloud_data_volume_name` (str, optional): The name of the cloud-based storage volume to use (e.g. "DataVolume3"). The volume must be created beforehand.
- `create_new_db` (bool, optional): Sets a flag which will re-initialize the database. All data and metadata is deleted during this action.
- `data_volume_id` (str, optional): Volume ID for the database data. data_volume_id can substitute data_volume_name. If both data_volume_id and data_volume_name are specified, data_volume_id is used instead.
- `data_volume_name` (str, optional): Volume name for the database data. The volume must be created beforehand. This parameter may be substituted with data_volume_id.
- `default_sys_passwd_hash` (str, optional): The hash used to set the default SYS password.
- `enable_auditing` (bool, optional): Enables or disables auditing for the database.
- `initial_sql` (str, optional): Absolute path to a YAML file with SQL statements to run during the first database startup.
- `interfaces` (str, optional): Comma-seperated list of network interfaces to use for the database. Leave empty to use all possible network interfaces.
- `jdbc_urls` (list|tuple, optional): 'List of URLs used to search for JDBC drivers, like: bucketfs://bfs2/bucket3/jdbcdir or file:///exa/etc/jdbc_drivers.'
- `ldap_server` (str, optional): Comma-separated list of LDAP Servers to use for remote database authentication, e.g. "ldap[s]://192.168.16.10". Each server must start with "ldap://" or "ldaps://".
- `max_system_heap_memory` (str, optional): Maximum heap memory for all processes in MiB or GiB, e.g. "64 GiB". The supported maximum is "128 GiB". You can specify "default" to unset a previously specified custom value and have the database fal
- `mem_size` (str, optional): Amount of memory (RAM) allocated to the database in MiB, GiB, or TiB, e.g. "2048 MiB".
- `oracle_url` (str, optional): 'An URL used to search for oracle inistant client, like: bucketfs://bfs2/bucket3/oracle_dir.'
- `owner` (list|tuple, optional): Tuple of (User id, group id) of the database owner in type integer or (user name, user group name).
- `params` (str|list, optional): A space-seperated list of extra database parameters which replaces the current settings. When used with params_add and/or params_delete, params is evaluated first. In general, parameters are in the fo
- `params_add` (str|list, optional): A space-seperated list of extra database parameters which are appended to current settings. When used with params and/or params_delete, params_add is evaluated second. In general, parameters are in th
- `params_delete` (str|list, optional): A space-separated list of extra database parameters to be deleted, such as "-param1" (to remove e.g. "-param1=42") or "-param2=10" (to remove "-param2=10", but not e.g. "-param2=20"). When used with p
- `port` (int, optional): Port that the database is reachable on.
- `version` (str, optional): Version of EXASolution executables.
- `volume_move_delay` (int, optional): Specifies the delay in seconds after which the volume on a failed node is moved to the reserve node automatically. When no value is set, there is no delay.
- `volume_quota` (str, optional): Maximum size of the data volume in MiB, GiB, or TiB. If the quota is reached, the database tries to shrink the volume on start-up if required and possible.

**Examples**:

```bash
confd_client db_configure {db_name: exa_db, mem_size: 2048MiB, owner: [1000, 1000], port: '3000', version: 8.0.0}
confd_client db_configure {db_name: exa_db, enable_auditing: true, params_add: [-forceProtocolEncryption=1]}
confd_client db_configure {db_name: exa_db, ldap_server: 'ldap://192.168.16.10:389'}
```

## db_enlarge

This job increases the number of active database nodes. The database must be
stopped to run this job. The nodes to be added must be added already as
reserve nodes. Afterwards, the database is started with the new nodes
automatically.

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `db_name` (str, required): The name of an existing database.
- `num_new_nodes` (str|int, required): Number of nodes which are to be added as active node to the given database. It is not possible to specify which nodes are added.

**Examples**:

```bash
confd_client db_enlarge {db_name: exa_db, num_new_nodes: 2}
```

## db_reset

This job resets the database. After performing this operation, the database
will be empty and all data (including metadata) is deleted. The database
must be offline to perform a reset.

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `db_name` (str, required): Name of an existing database to be reset.

**Examples**:

```bash
confd_client db_reset {db_name: DB1}
```

## db_resume_nodes

This job resumes the suspended nodes in the given database.

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `db_name` (str, required): Name of an existing database.
- `node_list` (list, required): 'List of suspended node names that should be resumed in the given database, for example: [''n11'', ''n12''].'

**Examples**:

```bash
confd_client db_resume_nodes {db_name: exa_db, node_list: [n12]}
```

## db_suspend_nodes

This job suspends the given nodes in the given database. Suspending a node
means that the node is marked as unavailable, even though it continues to
run. This can be used to perform maintenance on a node. If this is done to
an active node while the database is running, the database will shutdown and
restart with a reserve node instead (if configured).

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `db_name` (str, required): The name of an existing database.
- `node_list` (list, required): 'List of node names that should be suspended in the given database, for example: [''n11'', ''n12''].'

**Examples**:

```bash
confd_client db_suspend_nodes {db_name: exa_db, node_list: [n12]}
```

## db_add_reserve_nodes

**Adds reserve nodes to an Exasol database** for failover, high availability, and future database enlargement. Reserve nodes remain in **suspended state** until manually activated or used for automatic failover.

The nodes must already exist in the cluster (added with `node_add`) and be available (not assigned to another database).

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

## db_remove_reserve_nodes

This job removes the specified nodes as reserve nodes from the given
database. The specified nodes must already be added as reserve nodes.

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `db_name` (str, required): The name of an existing database.
- `node_list` (list, required): List of node IDs to be added as reserve nodes. The node IDs are integers.

**Examples**:

```bash
confd_client db_remove_reserve_nodes {db_name: exa_db, node_list: [12]}
```

## db_configure_kerberos

Sets parameters for DB Kerberos authentication.

    service: service_name}

**Permissions**: Users: root | Groups: root, exaadm, exadbadm

**Parameters**:

- `db_name` (str, required): The name of an existing database.
- `keytab` (str, required): Keytab file. Use {<filename} syntax of confd_client to pass the file name.
- `host` (str, optional): Host name of the Kerberos principal. Sets the -kerberosHostname DB parameter.
- `realm` (str, optional): Realm of the Kerberos principal. Sets the -kerberosRealm DB parameter.
- `service` (str, optional): Kerberos service name. Sets the -kerberosServiceName DB parameter.

**Examples**:

```bash
confd_client db_configure_kerberos {db_name: mydb, host: my_hostname, keytab: << keytab file contents >>, realm: EXAMPLE.COM,
```
