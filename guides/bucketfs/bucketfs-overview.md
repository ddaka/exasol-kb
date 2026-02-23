# BucketFS Overview

## What is BucketFS?

BucketFS is a synchronous file system that is available on all database nodes in an Exasol cluster. Each node in the cluster can connect to the BucketFS service and will see the same content as the other nodes.

A BucketFS service contains a number of buckets that can store a number of files. Each bucket can have different access privileges. Folders are not supported directly, but if you specify an upload path including folders, these will be created.

During installation, Exasol creates a BucketFS service named "bfsdefault" with a bucket named "default". You can create additional BucketFS services and buckets as needed after the installation.

In on-premises installations of Exasol 2025.1 and later, you can access BucketFS from [Exasol Admin](https://docs.exasol.com/db/latest/administration/on-premise/admin_interface/admin_ui_overview.htm). For earlier versions of Exasol we recommend that you use [BucketFS Client](https://docs.exasol.com/db/latest/database_concepts/bucketfs/bucketfs.htm#BucketFS) to manage buckets and files.

## Usage Notes

- **Data Replication**: The data in BucketFS is replicated locally on every server and automatically synchronized. To avoid performance issues, we recommend that you do not store very large amounts of data in BucketFS.

- **No Transactional Semantics**: In contrast to the database itself, BucketFS is a pure file-based system and has no transactional semantic. Writing data to BucketFS is an atomic operation, meaning that it either completes entirely or not at all. There is no lock on files, so the latest write operation will overwrite the file.

- **Backup**: The data in BucketFS is not part of the database backups and must be manually backed up by downloading the contained files.

- **Default Ports**: The default BucketFS service has HTTP deactivated and HTTPS activated on port 2581 by default. To change the ports in the BucketFS service, use the ConfD job [bucketfs_modify](https://docs.exasol.com/db/latest/confd/jobs/bucketfs_modify.htm).

## Additional References

- [Create New BucketFS Service](create-new-bucketfs-service.md)
- [Create New Bucket](create-new-bucket.md)
- [Manage Buckets and Files](manage-buckets-and-files.md)
- [Database Access in BucketFS](database-access-in-bucketfs.md)
- [Change Bucket Password](change-bucket-password.md)
