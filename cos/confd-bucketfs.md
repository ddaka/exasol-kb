---
tool_name: confd_client
doc_type: reference
category: BucketFS
subcommands: bucket_add, bucket_delete, bucket_modify, bucketfs_add, bucketfs_delete, bucketfs_info, bucketfs_list, bucketfs_modify
---

# confd_client — BucketFS Management

## Overview

Commands for managing BucketFS services and individual buckets: creating, deleting, modifying services and buckets.

All commands run inside the COS namespace (SSH port 20002).

## bucket_add

This job adds one or more buckets to a BucketFS service.

    write_password: cHc=}

**Permissions**: Users: root, _owner | Groups: root, exabfsadm, exaadm

**Parameters**:

- `bucket_name` (str, required): The bucket name.
- `bucketfs_name` (str, required): The name of the BucketFS sevice that will store the new bucket.
- `public` (bool, required): Indicates whether the bucket content is public. Valid values are True and False.
- `additional_files` (list|tuple, optional): 'List or tuple with additional files in the bucket. For example: [''EXAClusterOS:/opt/exasol/slc-*/ScriptLanguages-*''].'
- `read_password` (str, optional): The password for reading the bucket content.
- `write_password` (str, optional): The password for writing in the bucket.

**Examples**:

```bash
confd_client bucket_add {bucket_name: test_bucket, bucketfs_name: bucketfs1, public: false, read_password: cHc=,
```

## bucket_delete

This job deletes a bucket.

**Permissions**: Users: root, _owner | Groups: root, exabfsadm, exaadm

**Parameters**:

- `bucket_name` (str, required): The name of the bucket.
- `bucketfs_name` (str, required): The name of the BucketFS service.

**Examples**:

```bash
confd_client bucket_delete {bucket_name: test_bucket, bucketfs_name: bucketfs1}
```

## bucket_modify

This job modifies a bucket.

**Permissions**: Users: root, _owner | Groups: root, exabfsadm, exaadm

**Parameters**:

- `bucket_name` (str, required): The name of the bucket.
- `bucketfs_name` (str, required): The name of the BucketFS service that stores the bucket.
- `public` (bool, required): 'Indicates whether the bucket content is public. The following are valid values: True and False.'
- `additional_files` (list|tuple, optional): 'List or tuple with additional files in the bucket. For example: [''EXAClusterOS:/opt/exasol/slc-*/ScriptLanguages-*''].'
- `read_password` (str, optional): The password for reading the bucket content.
- `write_password` (str, optional): The password for writing in the bucket.

**Examples**:

```bash
confd_client bucket_modify {bucket_name: test_bucket, bucketfs_name: bucketfs1, public: false, write_password: cHc=}
```

## bucketfs_add

This job adds a BucketFS service.

**Permissions**: Users: root | Groups: root, exabfsadm, exaadm

**Parameters**:

- `bucketfs_name` (str, required): BucketFS service name.
- `http_port` (str|int, required): The http port of the bucketFS service. If the value for HTTPSPort is anything but 0, this parameter is ignored. To ensure http is deactivated, enter 0.
- `https_port` (str|int, required): The https port for the BucketFS service. To ensure the https connection is deactivated, enter 0 .
- `owner` (list|tuple, required): Tuple (User id, group id) of the database owner in type integer or (user name, user group name).
- `bucketvolume` (str, optional): The SDFS volume name.
- `mode` (str, optional): 'One of the following service modes: BucketFS, rsync or SDFS.'
- `path` (str, optional): Path to the BucketFS service.
- `sync_key` (str, optional): Synchronization key in base 64 format.
- `sync_period` (str|int, optional): Synchronization interval in seconds.

**Examples**:

```bash
confd_client bucketfs_add {bucketfs_name: bucketfs1, http_port: 0, https_port: 2581, owner: [500, 500]}
```

## bucketfs_delete

This job deletes a BucketFS service.

**Permissions**: Users: root, _owner | Groups: root, exabfsadm, exaadm

**Parameters**:

- `bucketfs_name` (str, required): BucketFS service name.

**Examples**:

```bash
confd_client bucketfs_delete {bucketfs_name: bucketfs1}
```

## bucketfs_info

This job provides information about a BucketFS service.

**Permissions**: Users: root | Groups: root, exabfsadm, exaadm

**Parameters**:

- `bucketfs_name` (str, required): The BucketFS service name.

**Examples**:

```bash
confd_client bucketfs_info {bucketfs_name: bucketfs1}
```

## bucketfs_list

This job lists BucketFS services.

  {}

**Permissions**: Users: root | Groups: root, exabfsadm, exaadm

## bucketfs_modify

This job modifies a BucketFS service.

**Permissions**: Users: root, _owner | Groups: root, exabfsadm, exaadm

**Parameters**:

- `bucketfs_name` (str, required): The name of the BucketFS service.
- `http_port` (str|int, optional): The http port of the bucketFS service. If the value for HTTPSPort is anything but 0, this parameter is ignored. To ensure http is deactivated, enter 0.
- `https_port` (str|int, optional): The https port for the BucketFS service. To ensure the https connection is deactivated, enter 0 .
- `owner` (list|tuple, optional): Tuple of (User id, group id) of the database owner in type integer or (user name, user group name).
- `sync_period` (str|int, optional): The synchronization period in seconds.

**Examples**:

```bash
confd_client bucketfs_modify {bucketfs_name: bucketfs1, http_port: 0, https_port: 2581, owner: [500, 500]}
```
