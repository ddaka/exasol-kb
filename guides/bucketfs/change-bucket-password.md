# Change Bucket Password

This article explains how to change the read/write passwords for a given bucket and BucketFS.

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

### Step 2: Change the Password

Use the ConfD job [bucket_modify](https://docs.exasol.com/db/latest/confd/jobs/bucket_modify.htm) to change the read and/or write password for a given bucket in the given BucketFS service. The password can be entered in either plain-text or a base64-encoded string.

**Example:**
```bash
confd_client bucket_modify \
  bucket_name: BUCKET_NAME \
  bucketfs_name: BUCKETFS_NAME \
  public: False \
  read_password: PASSWORD \
  write_password: PASSWORD
```

**Important:** Save the passwords in a secure location.

## Verification

You can use curl to perform a test read and write using the new passwords. For more details, see [Manage Buckets and Files in BucketFS](manage-buckets-and-files.md).

**Test read access:**
```bash
curl https://NODE_IP:BUCKETFS_PORT/BUCKET_NAME/ \
  --user r:NEW_READ_PASSWORD
```

**Test write access:**
```bash
echo "test" | curl -X PUT -T - https://NODE_IP:BUCKETFS_PORT/BUCKET_NAME/test.txt \
  --user w:NEW_WRITE_PASSWORD
```

## Additional References

- [BucketFS Overview](bucketfs-overview.md)
- [Create New Bucket](create-new-bucket.md)
- [Manage Buckets and Files](manage-buckets-and-files.md)
- [ConfD Documentation](https://docs.exasol.com/db/latest/confd/confd.htm)
