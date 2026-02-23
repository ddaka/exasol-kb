---
tool_name: confd_client
doc_type: guide
category: access-management
subcommands:
  - user_create
  - user_list
  - group_list
technical_entities:
  - users
  - groups
summary: >
  How to create system users in Exasol via confd_client — prerequisites,
  user_create parameters, and verification.
---

# Add Users

**Prerequisite:** You must be a member of the `exaadm` group.

## Procedure

1. Connect to COS:

```bash
c4 connect -i <PLAY_ID> -s cos
```

2. List available groups:

```bash
confd_client group_list
```

3. Create the user:

```bash
confd_client user_create \
  username: NEW_USER \
  userid: 1001 \
  password: NEW_PASSWORD \
  group: exausers \
  login_enabled: true
```

| Parameter       | Type    | Description                                    |
|-----------------|---------|------------------------------------------------|
| `username`      | string  | Name of the new user                           |
| `userid`        | integer | ID of the new user                             |
| `password`      | string  | Password for the new user                      |
| `group`         | string  | Group ID or name for the user's primary group  |
| `login_enabled` | boolean | Whether login is allowed (`true`/`false`)      |

## Verification

```bash
confd_client user_list
```
