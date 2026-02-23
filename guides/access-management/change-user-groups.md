---
tool_name: confd_client
doc_type: guide
category: access-management
subcommands:
  - user_modify
  - user_list
technical_entities:
  - users
  - groups
summary: >
  How to change user group membership in Exasol — modify primary group and
  add additional groups via confd_client user_modify.
---

# Change User Groups

When you create a user, you specify a primary group. You can later change the
primary group or add the user to additional groups.

**Prerequisite:** You must be a member of the `exaadm` group.

## Procedure

1. Connect to COS:

```bash
c4 connect -i <PLAY_ID> -s cos
```

2. Modify group membership:

```bash
confd_client user_modify \
  username: admin4 \
  group: exabfsadm \
  additional_groups: '[exastoradm, exausers]'
```

| Parameter           | Type   | Description                                         |
|---------------------|--------|-----------------------------------------------------|
| `username`          | string | Name of the user to modify                          |
| `group`             | string | New primary group (ID or name)                      |
| `additional_groups` | list   | Comma-separated list of additional group names       |

## Verification

```bash
confd_client user_list
```

Example output:

```
admin4:
  additional_groups:
    - exastoradm
    - exausers
  group: exabfsadm
  id: 1004
  login_enabled: true
```
