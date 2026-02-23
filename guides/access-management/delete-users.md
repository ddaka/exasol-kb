---
tool_name: confd_client
doc_type: guide
category: access-management
subcommands:
  - user_delete
  - user_list
technical_entities:
  - users
summary: >
  How to delete system users in Exasol via confd_client.
---

# Delete Users

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

3. Delete the user:

```bash
confd_client user_delete username: ADMIN2
```

## Verification

```bash
confd_client user_list
```

The user should no longer be listed.
