---
tool_name: confd_client
doc_type: guide
category: access-management
subcommands:
  - db_configure_kerberos
  - db_list
technical_entities:
  - Kerberos SSO
  - keytab
  - realm
summary: >
  How to configure Kerberos single sign-on (SSO) for Exasol databases via
  confd_client db_configure_kerberos — prerequisites, keytab upload, and
  realm/service/host restrictions.
---

# Kerberos SSO

Exasol supports single sign-on using Kerberos for JDBC and ODBC connections.

## Prerequisites

- Kerberos keytab file from KDC or Active Directory administrator
- Root access to the Exasol system

## Configure Kerberos

1. Connect to COS:

```bash
c4 connect -i <PLAY_ID> -s cos
```

2. Find the database name:

```bash
confd_client db_list
```

3. Configure Kerberos:

```bash
confd_client db_configure_kerberos \
  db_name: MY_DATABASE \
  keytab: '"{< ./my_keytab_file}"' \
  realm: EXAMPLE.COM \
  service: MY_SERVICE \
  host: MY_HOSTNAME
```

| Parameter   | Type   | Required | Description                                           |
|-------------|--------|----------|-------------------------------------------------------|
| `db_name`   | string | Yes      | Name of the Exasol database                           |
| `keytab`    | string | Yes      | Kerberos keytab file (`{<filename}` syntax)           |
| `realm`     | string | No       | Restrict to users from this realm only                |
| `service`   | string | No       | Restrict to this Kerberos service name                |
| `host`      | string | No       | Restrict to this hostname only                        |

To configure database users for Kerberos authentication, use `CREATE USER` or
`ALTER USER` SQL statements with the Kerberos principal.
