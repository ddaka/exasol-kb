# Database Access in BucketFS

This article explains how to access BucketFS from within the Exasol database.

## Overview

The Exasol database treats the BucketFS buckets as standard external data sources. Access control to the database is therefore established using a database `CONNECTION` object. The connection object contains the path to the bucket and the "read" password.

Passwords can be configured both when creating a bucket and on existing buckets. To learn more, see [Create New Bucket](create-new-bucket.md).

A publicly readable bucket is accessible to all users that can connect to the cluster, not just database users. We recommend that you do not make buckets publicly readable, and that you restrict read and write access to buckets using secure passwords.

## Creating a Connection to BucketFS

To make a bucket accessible to a database user, you must grant the connection to the user or a role of the user using the `GRANT CONNECTION` command.

**Example:**
```sql
CREATE CONNECTION my_bucket_access 
TO 'bucketfs:bfsdefault/bucket1'  
IDENTIFIED BY 'readpw';

GRANT CONNECTION my_bucket_access TO my_user;
```

## Accessing Files from the Database

When you have defined a connection to the bucket and granted it to a user, you can create a script that lists the files using a local path. The equivalent local path for a bucket follows this pattern: `/buckets/BUCKETFS_NAME/BUCKET_NAME`.

**Example: List Files in a Bucket**
```sql
--/
CREATE OR REPLACE PYTHON3 SCALAR SCRIPT "LS" ("my_path" VARCHAR(100)) EMITS (
"FILES" VARCHAR(100)) AS

import os

def run(ctx):
    for line in os.listdir(ctx.my_path):
        ctx.emit(line)
/

SELECT ls('/buckets/bfsdefault/bucket1');
```

**Output:**
```
FILES
---------------------
file1
tar1
```

**Example: List Files in a Subdirectory**
```sql
SELECT ls('/buckets/bfsdefault/bucket1/tar1/');
```

**Output:**
```
FILES
---------------------
a
b
```

## Archive File Handling

Archive files (`.zip`, `.tar`, `.tar.gz`, or `.tgz`) are always extracted automatically to enable scripts to access the contained files on the local file system. 

**Important Notes:**
- When you access the bucket from outside (using curl), you will only see the archive file.
- Locally (from within the database), you will use the extracted files.
- Both the archive file and the extracted files are stored, meaning storage space is required for both.
- If you want to work directly on an archive file without extraction, change the file extension so it is not recognized as an archive (for example, `.zipx` instead of `.zip`).

## Using BucketFS Files in UDFs

When you have established access to BucketFS from within the database, you can use the files in the buckets in your UDFs. For more information, see [Adding New Packages to Existing Script Languages](https://docs.exasol.com/db/latest/database_concepts/udf_scripts/adding_new_packages_script_languages.htm).

## Additional References

- [BucketFS Overview](bucketfs-overview.md)
- [Create New Bucket](create-new-bucket.md)
- [Manage Buckets and Files](manage-buckets-and-files.md)
