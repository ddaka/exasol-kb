---
tool_name: confd_client
doc_type: troubleshoot
category: database-sql
title: "Resolve non-public BucketFS access for virtual schema creation"
summary: "Fix virtual schema creation failures when adapter/JDBC JARs are stored in non-public BucketFS buckets."
---
# Resolve non-public BucketFS access for virtual schema creation

## Problem

Virtual schema creation fails because adapter or JDBC JAR files in BucketFS cannot be accessed by the database runtime.

Typical symptom: Java VM reports missing JAR file although the file exists in bucket listing.

## Root cause

Bucket permissions are non-public and no DB `CONNECTION` object is provided for BucketFS access.

## Procedure

### 1. Read BucketFS configuration

```shell
confd_client -c bucketfs_info -a 'bucketfs_name: bfsdefault'
```

Identify the encrypted `read_passwd` for the target bucket.

### 2. Decrypt read password

```shell
echo 'ENCRYPTED_PASSWORD_HERE' | openssl enc -base64 -d
```

Use secure shell history handling and do not expose plaintext credentials in logs.

### 3. Create DB connection object for BucketFS

```sql
CREATE OR REPLACE CONNECTION bucketfs_access
TO 'bucketfs:bfsdefault/default'
IDENTIFIED BY 'DECRYPTED_READ_PASSWORD';
```

### 4. Re-run virtual schema creation

Retry the virtual schema DDL using the bucket-access connection as required by the adapter setup.

## Validation

- Virtual schema DDL succeeds.
- Adapter can load all required JAR files.
- Queries against virtual schema execute without classpath/file-not-found errors.

## Result

Non-public bucket access is explicitly granted through a DB connection object, allowing virtual schema components to load correctly.

---

_We welcome feedback on this troubleshooting article._
