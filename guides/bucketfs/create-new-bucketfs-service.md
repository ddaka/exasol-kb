# Create New BucketFS Service

Each configured data disk in Exasol has a preinstalled default BucketFS service named bfsdefault. The default BucketFS service includes a default bucket that contains preinstalled script languages (Java, Python, and R). You can create and configure any number of buckets within the default BucketFS service.

If you want to make files available on different ports, or if you want to store larger files in BucketFS, we recommend that you create a new BucketFS service.

## Prerequisites

There are no prerequisites for this procedure.

## Procedure

This procedure uses the command-line tool confd_client, which is available on all database nodes. For more information, see [ConfD](https://docs.exasol.com/db/latest/confd/confd.htm).

Placeholder values are indicated with UPPERCASE characters. Replace the placeholders with your own values.

### Step 1: Connect to COS

Connect to the cluster operating system (COS) using `c4 connect -i PLAY_ID -s cos`.

**Example:**
```bash
./c4 connect -i c3275f84 -s cos
```

To find the play ID, you can use `c4 ps`.

For more information about how to use c4 commands, see [How to use c4](https://docs.exasol.com/db/latest/administration/on-premise/c4/using_c4.htm#Connecttoadeployment).

### Step 2: Create the BucketFS Service

To create a BucketFS service, use the ConfD job [bucketfs_add](https://docs.exasol.com/db/latest/confd/jobs/bucketfs_add.htm) with the following parameters:

| Parameter | Type | Description |
|-----------|------|-------------|
| bucketfs_name | string | The name of the new BucketFS service. |
| http_port | string, integer | The HTTP port that the BucketFS service should be accessible on. To disable HTTP, enter 0. This parameter is ignored if the value for https_port is anything else than 0. |
| https_port | string, integer | The HTTPS port for the BucketFS service. To disable HTTPS, enter 0. |
| owner | tuple, list | The database owner as an integer tuple (user id, group id) or list (user name, user group name). |

**Example:**
```bash
confd_client bucketfs_add bucketfs_name: mybucketfs http_port: 0 https_port: 2582 owner: [500,500]
```

## Verification

To verify that the BucketFS service was created, use the ConfD job [bucketfs_list](https://docs.exasol.com/db/latest/confd/jobs/bucketfs_list.htm).

**Example:**
```bash
confd_client bucketfs_list
```

**Output:**
```
- bfsdefault
- mybucketfs
```

## Next Steps

When the BucketFS service has been created, you can add buckets to store files or data. For more information, see [Create New Bucket](create-new-bucket.md).

## Additional References

- [BucketFS Overview](bucketfs-overview.md)
- [ConfD Documentation](https://docs.exasol.com/db/latest/confd/confd.htm)
