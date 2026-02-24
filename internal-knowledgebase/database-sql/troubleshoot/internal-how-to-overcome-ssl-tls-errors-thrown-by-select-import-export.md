---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: database-sql
title: "Internal: overcome SSL/TLS errors in SELECT/IMPORT/EXPORT paths"
summary: "Troubleshooting playbook for PKIX/self-signed certificate errors when Exasol components act as TLS clients."
---
# Internal: overcome SSL/TLS errors in SELECT/IMPORT/EXPORT paths

## Problem

Statements executed from inside Exasol can fail with TLS trust errors, even when SQL client connectivity works.

Typical messages:

- `PKIX path building failed`
- `unable to find valid certification path to requested target`
- `TLS connection ... failed: self signed certificate`

Root cause: the runtime used by Exasol for that operation does not trust the remote certificate chain.

## Key principle

The truststore to update depends on the component that performs the outbound connection:

- `IMPORT FROM JDBC`: Java truststore used by loader/runtime Java.
- `IMPORT FROM EXA`: OS truststore used by Client SDK/C++ path.
- Java UDF / adapters: Java truststore inside Script Language Containers (SLC).

## 1) IMPORT FROM JDBC (generic fix)

Import the remote certificate into Java truststore on data nodes.

Typical truststore path:

```text
/etc/pki/ca-trust/extracted/java/cacerts
```

Command:

```shell
keytool -keystore /etc/pki/ca-trust/extracted/java/cacerts \
  -import -file <certificate.pem> -alias <alias_name> \
  -storepass changeit -noprompt
```

For clusters, distribute certificate and run on all relevant nodes.

## 2) IMPORT FROM JDBC (driver-specific fix)

Some JDBC drivers support explicit keystore/truststore parameters in JDBC URL.

Approach:

1. Build keystore containing remote cert.
1. Upload keystore file to BucketFS.
1. Pass driver-specific keystore parameters in JDBC URL.

Use only when driver supports it and path resolution is validated in your environment.

## 3) IMPORT FROM EXA

`IMPORT FROM EXA` uses OS truststore, not Java truststore.

CentOS-style update:

```shell
cp <certificate.pem> /etc/pki/ca-trust/source/anchors/
update-ca-trust
```

Ubuntu-style update:

```shell
cp <certificate.crt> /usr/local/share/ca-certificates/
update-ca-certificates
```

Apply on required nodes in COS namespace.

## 4) Java UDF and Java-based adapters

Java UDF runtime uses Java truststores in SLC images/paths.

Search and update all relevant `cacerts` files, then sync:

```shell
for SLC in $(find /d02_data /exa/data /opt/exasol -type f -regex '.*/ScriptLanguages.*/java/cacerts'); do
  keytool -keystore "$SLC" -import -file <certificate.pem> -alias <alias_name> -storepass changeit -noprompt
  cos_sync_files "$SLC"
done
```

Run on an active node.

## Validation

After certificate import:

1. Re-run failing statement.
1. Verify no PKIX/self-signed errors remain.
1. Confirm trust entry with `keytool -list` where applicable.

## Operational caveats

- Manual truststore changes may be overwritten by OS or DB updates.
- New SLC versions may require certificate re-import.
- Use descriptive aliases and track applied certificates in change records.

## Result

Issue is resolved when outbound TLS handshake succeeds for the specific runtime path (JDBC loader, EXA import path, or UDF runtime).

---

_We welcome feedback on this troubleshooting article._
