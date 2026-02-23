---
tool_name: confd_client
doc_type: reference
category: access-management
subcommands:
  - group_list
technical_entities:
  - users
  - groups
summary: >
  Exasol system user groups — defines which administrative tasks each group
  can perform via confd_client.
---

# User Groups

User groups determine which administrative tasks (ConfD jobs) a user can
perform. Available groups:

| Group        | Description                      |
|--------------|----------------------------------|
| `exaadm`     | Exasol system administrator      |
| `exabfsadm`  | BucketFS administrator           |
| `exadbadm`   | Exasol database administrator    |
| `exastoradm` | Exasol storage administrator     |
| `exausers`   | Exasol users                     |
| `root`       | Root user                        |

> **Note:** System administration users are **not** the same as database users.

## List Groups

```bash
confd_client group_list
```
