---
tool_name: confd_client
doc_type: guide
category: system
title: "SaaS - Generate wildcard certificates for customer databases with letsencrypt and certbot"
summary: "Generate wildcard TLS certificates with certbot and apply them to SaaS customer databases via confd cert_update."
---
# SaaS - Generate wildcard certificates for customer databases with letsencrypt and certbot

## Purpose

Create or renew wildcard certificates for SaaS domains and propagate them to customer databases.

## Prerequisites

- `certbot` installed on the management node.
- DNS write access for ACME TXT challenge (typically Route53).
- Access to SaaS management node filesystem and confd tooling.

## Generate wildcard certificate

1. Start manual DNS challenge flow:

```bash
sudo certbot certonly --manual --preferred-challenges dns -d '*.clusters-dev.exasol.com'
```

2. Create the requested `_acme-challenge` TXT record and wait for DNS propagation.

3. After successful issuance, copy certificate material to SaaS TLS location:

```bash
sudo mkdir -p /exa/etc/ssl/saas
sudo cp /etc/letsencrypt/live/<cert-name>/fullchain.pem /exa/etc/ssl/saas/fullchain.pem
sudo cp /etc/letsencrypt/live/<cert-name>/privkey.pem /exa/etc/ssl/saas/privkey.pem
sudo cp /etc/letsencrypt/live/<cert-name>/cert.pem /exa/etc/ssl/saas/cert.pem
sudo cp /etc/letsencrypt/live/<cert-name>/chain.pem /exa/etc/ssl/saas/chain.pem
```

4. Apply secure permissions:

```bash
sudo chmod 600 /exa/etc/ssl/saas/privkey.pem
sudo chmod 644 /exa/etc/ssl/saas/cert.pem /exa/etc/ssl/saas/chain.pem /exa/etc/ssl/saas/fullchain.pem
```

## Update certificate on existing customer databases

1. Record initial customer DB and worker-cluster states.
2. Ensure customer DB and worker clusters are running for certificate update.
3. Get `platform_reference` from `saas_db_info`.
4. Run `cert_update` on that platform:

```bash
params="{cert: \"{< /exa/etc/ssl/saas/fullchain.pem}\", key: \"{< /exa/etc/ssl/saas/privkey.pem}\"}"
confd_client -s <platform_reference> -c cert_update -a "$params"
```

5. After successful update, stop customer DB.
6. Restart `c4_cloud_command` on access node (`n10`):

```bash
sudo systemctl restart c4_cloud_command.service --no-block
```

7. Restore DB/cluster to the same running/stopped state captured in step 1.

## Notes

- Manual DNS challenge certificates do not auto-renew unless hook automation is implemented.
- Track certificate expiry proactively.

## Canonical references (de-duplication)

- `documents/cos/confd-system-and-infrastructure.md` (`cert_update`)
- `documents/cos/cos-sync-files.md` (cluster-wide file sync pattern)
- `documents/internal-knowledgebase/system/guide/how-to-use-confd-client-command-to-interact-with-saas.md`
