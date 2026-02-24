---
tool_name: confd_client
doc_type: guide
category: system
title: "How to use Comodo certificate"
summary: "Apply Comodo TLS certificate material for SaaS environments and update existing databases via cert_update."
---
# How to use Comodo certificate

## Scope

SaaS certificate rollout for management/customer database TLS.

## Notes

- Applies to newly created databases.
- COS version handling differs (`>= 8.42.0` vs older versions).

## 1) Prepare certificate files in SaaS SSL directory

For COS `>= 8.42.0`:

```shell
mkdir -p /exa/etc/ssl/saas_comodo
cd /exa/etc/new_cert
cp STAR_clusters-staging_exasol_com.crt /exa/etc/ssl/saas_comodo/ssl.crt
cp STAR_clusters-staging_exasol_com.ca-bundle /exa/etc/ssl/saas_comodo/ssl.ca
cp server.key /exa/etc/ssl/saas_comodo/ssl.key
mv /exa/etc/ssl/saas /exa/etc/ssl/saas_backup
cp -r /exa/etc/ssl/saas_comodo /exa/etc/ssl/saas
```

For older COS versions:

```shell
mkdir -p /exa/etc/ssl/saas_comodo
cd /exa/etc/new_cert
cp server.key /exa/etc/ssl/saas_comodo/privkey.pem
cat STAR_clusters-staging_exasol_com.crt STAR_clusters-staging_exasol_com.ca-bundle > /exa/etc/ssl/saas_comodo/fullchain.pem
mv /exa/etc/ssl/saas /exa/etc/ssl/saas_backup
cp -r /exa/etc/ssl/saas_comodo /exa/etc/ssl/saas
```

## 2) Update existing databases

```shell
params="{cert: \"{< /exa/etc/new_cert/STAR_clusters-staging_exasol_com.crt}\", key: \"{< /exa/etc/new_cert/server.key}\", ca: \"{< /exa/etc/new_cert/STAR_clusters-staging_exasol_com.ca-bundle}\"}"
confd_client -s <deployment_id> -R <region> -c cert_update -a "$params"
```

## Validation

- Verify certificate files exist with correct ownership/permissions.
- Verify DB TLS handshake uses updated certificate chain.

## De-duplication note

Canonical `cert_update` command reference is maintained in:

- `documents/cos/confd-system-and-infrastructure.md`


