---
tool_name: confd_client
doc_type: guide
category: system
title: "How to create a BucketFS and buckets on SaaS"
summary: "Create BucketFS and buckets on SaaS customer deployments using confd_client, then validate object access with curl."
---
# How to create a BucketFS and buckets on SaaS

## Purpose

Create a new BucketFS service and bucket on a SaaS customer deployment when UI-based management is unavailable.

## Prerequisites

- Customer database deployment is running.
- Access to customer instance/container shell.
- `confd_client` access with required privileges.

## 1) Connect to target environment

Connect to the target instance/container using the standard SaaS access workflow.

## 2) Create BucketFS

```shell
confd_client -c bucketfs_add -a '{"bucketfs_name":"newbucket-bucketfs","http_port":"6932","https_port":"6933","owner":[500,500]}'
```

Expected side effect: updates BucketFS config files (for example `/exa/etc/bucketfs.cfg`, `/exa/etc/bucketfs_db.cfg`).

## 3) Create bucket inside BucketFS

```shell
confd_client -c bucket_add -a '{"bucket_name":"newbucketfs","bucketfs_name":"newbucket-bucketfs","public":true,"read_password":"<BASE64_READ_PASSWORD>","write_password":"<BASE64_WRITE_PASSWORD>"}'
```

## 4) Verify EXAConf/BucketFS config

```shell
grep -n "\[BucketFS\|\[Bucket" /exa/etc/EXAConf
```

Confirm the new BucketFS and bucket entries exist.

## 5) Validate bucket access with curl

Use decoded read/write credentials for HTTP basic auth.

Upload file:

```shell
curl -X PUT -T testfile.jar http://w:<write_password>@<public_ip>:6932/newbucketfs-bucket/testfile.jar
```

List bucket content:

```shell
curl http://r:<read_password>@<public_ip>:6932/newbucketfs-bucket/
```

Delete file:

```shell
curl -X DELETE http://w:<write_password>@<public_ip>:6932/newbucketfs-bucket/testfile.jar
```

## Notes

- Keep bucket credentials in secure secret storage.
- Prefer private buckets unless public access is explicitly required.

## Reference

- <https://exasol.my.site.com/s/article/Exasol-on-Docker-How-to-Create-a-BucketFS-and-Buckets-Inside>


