---
tool_name: confd_client
doc_type: guide
category: access-management
subcommands:
  - cert_update
  - db_info
technical_entities:
  - TLS certificate
  - certificate chain
  - fingerprint
summary: >
  How to upload and manage TLS certificates in Exasol — create a certificate
  chain, upload via confd_client cert_update, and retrieve the TLS fingerprint.
---

# Upload TLS Certificate

TLS certificates encrypt communication between Exasol and administration tools,
database clients, and BucketFS.

## Prerequisites

- Self-signed or CA-signed server certificate (RSA or ECC)
- Private key file (max 8192 bits, **not** password-encrypted)
- Root certificate and any intermediate certificates

## Create a Certificate Chain

A certificate chain is a text file containing certificates from server to root:

```bash
# Copy server certificate
cat server_cert.crt > cert_chain.pem

# Append root certificate (note: >> not >)
cat ca_cert.crt >> cert_chain.pem
```

### Hostnames in the Server Certificate

If the SQL driver uses `first_host..last_host:port`, the exact hostnames must be
in the CN or SAN of the server certificate.

| Connection String Format                      | Required in Certificate                |
|-----------------------------------------------|----------------------------------------|
| `10.11.12.13..15` (IP range)                  | 10.11.12.13, 10.11.12.14, 10.11.12.15 |
| `10.11.12.13,10.11.12.16` (IP list)           | 10.11.12.13, 10.11.12.16              |
| `db-node1..3.mydomain` (DNS range)            | Exact names or `*.mydomain`            |

## Upload Certificate and Key

1. Copy files to COS:

```bash
tar cf - cert_chain.pem key.pem | c4 connect -i <PLAY_ID> -n <NODE> -s cos tar xvf -
```

2. Connect to COS:

```bash
c4 connect -i <PLAY_ID> -s cos
```

3. Upload using ConfD:

```bash
confd_client cert_update cert: '"{< ./cert_chain.pem}"' key: '"{< ./key.pem}"'
```

> Only one TLS certificate can be active at a time. Uploading a new certificate
> overwrites the previous one.

## Enable the Certificate

SQL connections established after upload automatically use the new certificate.
A database restart is **not required** unless:

- The database uses non-default certificate paths
- The legacy TLS library is enabled

## TLS Fingerprint

For clients that don't support certificate verification, use fingerprints:

```bash
c4 connect -i <PLAY_ID> -s cos -- 'confd_client db_info db_name: MY_DATABASE'
```

Output includes:

```
certificate fingerprint: 3C16E3CB7D1D1058F2675B92FEB7B91F6D54875EBB17019028FA6F27C714C555
```

> Uploading a new certificate generates a new fingerprint — update all connection
> strings accordingly.
