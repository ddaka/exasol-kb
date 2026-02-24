---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: database-sql
title: "Virtual Schema TLS Errors with Internal Root CA Certificates"
summary: "Troubleshooting guide for PostgreSQL Virtual Schema SSL failures when using internal CA certificates and BucketFS paths."
---

# Virtual Schema TLS Errors with Internal Root CA Certificates

## Scope

This article covers PostgreSQL Virtual Schema connection failures in Exasol when TLS is enabled and certificates are signed by an internal CA.

## Common Errors

### 1. Network/TLS timeout

```text
java.net.SocketTimeoutException: Read timed out
```

### 2. Certificate path validation failure

```text
unable to find valid certification path to requested target
```

### 3. Certificate file path not accessible

```text
Could not open SSL root certificate file /d02_data/bfsdefault/bucket1/ca-certificates.pem
```

## Diagnostic Approach

1. Verify network path and firewall rules between Exasol and PostgreSQL.
2. Validate certificate chain and CA bundle content.
3. Validate that `sslrootcert` path is accessible in the execution namespace used by the adapter.

## Connection Examples

Default Java SSL factory example:

```sql
CREATE CONNECTION JDBC_TEST_CONN
TO 'jdbc:postgresql://postgresql-data:5432/pods?sslmode=verify-ca&sslrootcert=/buckets/bfsdefault/bucket1/ca-certificates.pem&sslfactory=org.postgresql.ssl.DefaultJavaSSLFactory';
```

LibPQ factory example:

```sql
CREATE CONNECTION JDBC_SSL_CONN
TO 'jdbc:postgresql://postgresql-data:5432/pods?sslmode=verify-ca&sslrootcert=/buckets/bfsdefault/bucket1/ca-certificates.pem&sslfactory=org.postgresql.ssl.LibPQFactory';
```

## Root Cause Pattern: Namespace Path Mismatch

A frequent failure pattern is that one component expects BucketFS-style path (`/buckets/...`) while another resolves only real node filesystem paths (`/d02_data/...`).

Because Virtual Schema adapter and import/runtime components may operate in different namespaces, one absolute certificate path may not work for both.

## Workaround

Create a path bridge (symlink) so the expected BucketFS-style path resolves correctly:

```bash
mkdir -p /buckets/bfsdefault/bucket1
cd /buckets/bfsdefault/bucket1
ln -s ../../../d02_data/bfsdefault/bucket1/ca-certificates.pem ca-certificates.pem
```

Run this on all relevant data nodes, including reserve nodes.

Then recreate/retest the connection and Virtual Schema.

## Alternate Compatibility Setting

If policy allows and verification mode differs in your environment:

```sql
CREATE CONNECTION JDBC_SSL_CONN
TO 'jdbc:postgresql://postgresql-data:5432/pods?sslmode=require&sslrootcert=/buckets/bfsdefault/bucket1/ca-certificates.pem&sslfactory=org.postgresql.ssl.LibPQFactory';
```

## Notes

- Prefer certificate verification modes aligned with your security policy.
- Keep CA bundle deployment and path mapping consistent across all nodes.
