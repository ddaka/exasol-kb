---
tool_name: internal-knowledgebase
doc_type: guide
category: database-sql
title: "LDAP user authentication with Exasol"
summary: "Configure LDAP authentication, create LDAP-mapped users, and tune LDAP timeout behavior."
---
# LDAP user authentication with Exasol

## Purpose

Configure Exasol database authentication against an LDAP directory service.

## Configure LDAP server for a database

1. Stop the database.
1. In EXAoperation, edit database properties and configure LDAP server endpoint.
1. Start the database.

## Create LDAP-authenticated users

Create users mapped to LDAP distinguished names (DN):

```sql
CREATE USER <username> IDENTIFIED AT LDAP AS '<distinguished_name>';
```

To discover DN, use `ldapsearch` (example):

```shell
ldapsearch -h 10.70.19.10 \
  -b "CN=Users,DC=ldaptest,DC=exasol,DC=com" \
  -v -x \
  -D "testuser@ldaptest.exasol.com" \
  -w <password> | grep distinguishedName | grep testuser
```

## Use LDAP proxy for multi-directory setups

Exasol supports one configured LDAP endpoint per database. For environments that require multiple LDAP directories, configure an LDAP proxy as the single endpoint:

1. Stop the database.
1. Configure LDAP proxy address in database properties.
1. Start the database.

Existing LDAP users continue to work if DN mapping remains valid.

## Resolve LDAP timeout issues

Example log symptom:

```text
LDAP bind failed: Timed out (LDAPTimeoutInSeconds = 5)
```

Default timeout is 5 seconds. Increase when network/LDAP response time requires it.

1. Stop the database.
1. Set extra DB parameter in EXAoperation, for example:

```text
-LDAPTimeoutInSeconds=10
```

1. Start the database.

## Validation

- Test login with a mapped LDAP user.
- Check DB logs for successful bind/authentication.
- Confirm timeout-related warnings are no longer present.


