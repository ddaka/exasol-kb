---
tool_name: confd_client
doc_type: guide
category: access-management
subcommands:
  - user_passwd
  - user_list
technical_entities:
  - users
  - password
summary: >
  How to change a system user password in Exasol via confd_client user_passwd.
---

# Change a Password

**Prerequisite:** You must be a member of the `exaadm` group.

## Procedure

1. Connect to COS:

```bash
c4 connect -i <PLAY_ID> -s cos
```

2. Find the user:

```bash
confd_client user_list
```

3. Change the password:

```bash
confd_client user_passwd username: USERNAME password: NEW_PASSWORD
```

The password is encoded by default.

## Verification

Log in using the new password.
