# Create New Bucket

This article explains how to create a bucket in a BucketFS service.

## Prerequisites

The BucketFS service must exist.

## Procedure

This procedure uses the command-line tool confd_client, which is available on all database nodes. For more information, see [ConfD](https://docs.exasol.com/db/latest/confd/confd.htm).

Placeholder values are indicated with UPPERCASE characters. Replace the placeholders with your own values.

To create a bucket, use the ConfD job [bucket_add](https://docs.exasol.com/db/latest/confd/jobs/bucket_add.htm) with the following parameters:

| Parameter | Type | Description |
|-----------|------|-------------|
| bucket_name | string | The name of the new bucket. |
| bucketfs_name | string | The name of the BucketFS service that will store the new bucket. This can be either the default service "bfsdefault" or a service that you created. See also Create New BucketFS Service. |
| public | boolean | Indicates whether the bucket content should be public. Valid values are True and False. |
| read_password | string | Password for read permissions in the bucket. |
| write_password | string | Password for read+write permissions in the bucket. |

**Example:**
```bash
confd_client bucket_add bucket_name: my_bucket bucketfs_name: my_bucketfs public: False read_password: READPW write_password: WRITEPW
```

## Verification

To verify that the bucket was successfully created, you can use the ConfD job [bucketfs_info](https://docs.exasol.com/db/latest/confd/jobs/bucketfs_info.htm).

This ConfD job returns the full details about the specified BucketFS service including all buckets. If you only want to verify that the newly created bucket exists, you can append the command with `| grep <bucket name>`.

**Example:**
```bash
confd_client bucketfs_info bucketfs_name: my_bucketfs | grep my_bucket
```

**Output:**
```
  my_bucket:
    _sec_name: 'Bucket : my_bucket'
    name: my_bucket
```

## Next Steps

For information about how to access and use the new bucket, see [Manage Buckets and Files in BucketFS](manage-buckets-and-files.md).

For information about how to expand scripting languages using BucketFS, see [Adding New Packages to Existing Script Languages](https://docs.exasol.com/db/latest/database_concepts/udf_scripts/adding_new_packages_script_languages.htm).

## Additional References

- [BucketFS Overview](bucketfs-overview.md)
- [Create New BucketFS Service](create-new-bucketfs-service.md)
- [ConfD Documentation](https://docs.exasol.com/db/latest/confd/confd.htm)
