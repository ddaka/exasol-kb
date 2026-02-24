---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "Upload EXAoperation Certificate via Shell (Legacy)"
summary: "Legacy procedure for uploading custom EXAoperation TLS certificates on pre-6.0 environments using shell access and cluster file synchronization."
---

# Upload EXAoperation Certificate via Shell (Legacy)

## Status

This procedure applies to older EXAoperation versions (pre-6.0) and is obsolete for modern versions.

## Context

Historically, custom TLS certificate deployment for EXAoperation required manual shell steps.

Related changelog entry:

- <https://exasol.my.site.com/s/article/Changelog-content-3371?language=en_US>

## Prerequisites

- Root shell access to the cluster.
- Certificate/key files available on the host:
  - `server_crt.pem`
  - `server_key.pem`
- Certificate should use SHA-256.

## Procedure

### 1. Upload certificate files to the host

Copy `server_crt.pem` and `server_key.pem` to the target host (for example into root home directory).

### 2. Stop EXAoperation appserver process

```bash
cosrm -a "$(cosps | awk '/appserverd/{ print $1; }')"
```

### 3. Replace certificate files

```bash
mv ~/server_crt.pem "$COS_DIRECTORY/var/exaoperation/inst/etc/server_crt.pem"
mv ~/server_key.pem "$COS_DIRECTORY/var/exaoperation/inst/etc/server_key.pem"
```

### 4. Synchronize files across nodes

```bash
cos_sync_files "$COS_DIRECTORY/var/exaoperation/inst/etc/server_crt.pem"
cos_sync_files "$COS_DIRECTORY/var/exaoperation/inst/etc/server_key.pem"
```

### 5. Start EXAoperation appserver

```bash
cosexec --single-instance --auto-restart --auto-add -- "$COS_DIRECTORY/libexec/appserver"
```

## Notes

- Prefer built-in certificate management features in newer EXAoperation versions.
- Validate certificate chain and key pairing before restart.
