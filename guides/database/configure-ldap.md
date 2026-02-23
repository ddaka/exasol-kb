---
tool_name: confd_client
doc_type: guide
category: database-management
subcommands:
  - db_configure
  - db_info
technical_entities:
  - LDAP
  - database
summary: >
  How to configure an LDAP server for Exasol database authentication via
  confd_client db_configure with the ldap_server parameter.
---

# Configure LDAP Server

**Prerequisites:** Database must be stopped. Nodes must be able to reach the
LDAP server.

## Procedure

```bash
confd_client db_configure db_name: MY_DATABASE \
  ldap_server: '"ldap://192.168.16.10:389, ldap://192.168.16.11:389"'
```

- URL must start with `ldap://` or `ldaps://`
- Multiple servers as comma-separated list (tried in order)
- Only one LDAP server is used for authentication at a time

## Verification

```bash
confd_client db_info db_name: MY_DATABASE
```
