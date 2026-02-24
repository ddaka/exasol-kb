---
tool_name: internal-knowledgebase
doc_type: guide
category: database-sql
title: "Loading CSV/FBV files from S3-compatible storage"
summary: "Internal feature note for loading CSV/FBV files through S3-style authentication against non-standard S3 endpoints."
---
# Loading CSV/FBV files from S3-compatible storage

## Purpose

Document the internal feature behavior that allows S3 authentication for non-default S3 URL patterns.

## Background

Public documentation for CSV/FBV loading covers standard endpoints, but some customer setups require S3-compatible endpoints or proxies.

An internal enhancement was implemented in [SPOT-7927](https://exasol.atlassian.net/browse/SPOT-7927) to support this scenario.

## Enablement requirements

To activate this behavior:

1. Prefix the URL scheme with `s3-` before `http` or `https`.
2. Configure at least one domain in `-etlS3DomainList` (comma-separated for multiple domains).

Bucket parsing rule:
- Everything after `http[s]://` and before `.<domain>` is interpreted as the bucket name.

## Example

```sql
EXPORT (SELECT 1)
INTO CSV AT 's3-https://testbucket.s3.amazonaws.com'
USER 'X' IDENTIFIED BY 'Y'
FILE 'test.csv';
```

Domain list setting:

```text
-etlS3DomainList=s3.amazonaws.com
```

## Important notes

- This feature was introduced for a specific customer proxy scenario.
- This behavior is internal and not broadly advertised.
- Verified test coverage was done with Amazon S3 buckets. Compatibility with third-party S3 implementations is not guaranteed.

## References

- [Load data from CSV/FBV files](https://docs.exasol.com/db/latest/loading_data/csv_fbv_file_types.htm)
- [SPOT-7927: allow S3 authentication for non-S3 URLs via `s3-` prefix](https://exasol.atlassian.net/browse/SPOT-7927)
