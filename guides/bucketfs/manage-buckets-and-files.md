# Manage Buckets and Files in BucketFS

This section explains how to access and manage buckets and files in BucketFS.

## How to Access BucketFS

You can access files in a BucketFS bucket from outside the cluster by using an HTTPS client and providing the IP address of one of the data nodes, the BucketFS port, the bucket name, and the read/write passwords. The read and write passwords should be set when you create the bucket. To configure passwords for an existing bucket, use the ConfD job [bucket_modify](https://docs.exasol.com/db/latest/confd/jobs/bucket_modify.htm).

**Key Points:**
- The write password allows both read and write access.
- If a bucket is not publicly readable, you can only access it using the configured read/write passwords.
- Your network must allow traffic on the BucketFS port. See also [Firewall and Port Settings](https://docs.exasol.com/db/latest/administration/on-premise/manage_network/system_network_settings.htm#top).

## Clients and Tools

In on-premises installations of Exasol 2025.1 and later, you can upload and manage files in BucketFS using Exasol Admin. In other versions of Exasol we recommend that you use [BucketFS Client](https://docs.exasol.com/db/latest/database_concepts/bucketfs/bucketfs.htm#BucketFS), which is a Java-based tool that runs on Windows and Linux. You can also upload and manage files from the command line using [curl](https://curl.se/).

### Using curl

You can use curl to upload, download, and delete files in BucketFS buckets.

**Upload a file:**
```bash
curl -X PUT -T /path/to/local/file.txt https://NODE_IP:BUCKETFS_PORT/BUCKET_NAME/file.txt \
  --user w:WRITE_PASSWORD
```

**Download a file:**
```bash
curl https://NODE_IP:BUCKETFS_PORT/BUCKET_NAME/file.txt \
  --user r:READ_PASSWORD \
  -o /path/to/local/file.txt
```

**Delete a file:**
```bash
curl -X DELETE https://NODE_IP:BUCKETFS_PORT/BUCKET_NAME/file.txt \
  --user w:WRITE_PASSWORD
```

**List files in a bucket:**
```bash
curl https://NODE_IP:BUCKETFS_PORT/BUCKET_NAME/ \
  --user r:READ_PASSWORD
```

## Additional References

- [BucketFS Overview](bucketfs-overview.md)
- [Manage files using Exasol Admin](https://docs.exasol.com/db/latest/administration/on-premise/bucketfs/bucketfs_file_access_adminui.htm)
- [Manage files using curl](https://docs.exasol.com/db/latest/administration/on-premise/bucketfs/bucketfs_file_access_curl.htm)
- [Database Access in BucketFS](database-access-in-bucketfs.md)
