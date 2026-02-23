---
tool_name: confd_client
doc_type: reference
category: User and Group Management
subcommands: user_create, user_delete, user_info, user_list, user_modify, user_passwd, group_create, group_delete, group_info, group_list
---

# confd_client — User and Group Management

## Overview

Commands for managing OS-level users and groups on the Exasol cluster: creating, deleting, modifying users and groups, changing passwords.

All commands run inside the COS namespace (SSH port 20002).

## user_create

This job creates a new user and syncs the change on all nodes.

    username: user_1}

**Permissions**: Users: root | Groups: root, exaadm

**Parameters**:

- `group` (str|int, required): 'Group ID or group name for the group that the new user should belong to. Example: exausers.'
- `login_enabled` (bool, required): Boolean value defining if login is allowed (true) or not (false).
- `password` (str, required): Password for the new user.
- `userid` (int, required): 'ID of the new user in integer format. Example: 1001.'
- `username` (str, required): 'Name of the new user. Example: user_1.'
- `additional_groups` (list, optional): 'Comma-separated list of group names of additional groups that the new user should be a member of. Example: [root, exausers].'
- `authorized_keys` (list, optional): A list of authorized keys for the new user.
- `encode_passwd` (bool, optional): Boolean value defining if the password should be encoded (true) or not encoded (false).

**Examples**:

```bash
confd_client user_create {group: exausers, login_enabled: true, password: secret_password, userid: 1001,
```

## user_delete

This job deletes a given user (identified by username) and syncs the changes
on all nodes.

**Permissions**: Users: root | Groups: root, exaadm

**Parameters**:

- `username` (str, required): String containing the name of the user to delete.

**Examples**:

```bash
confd_client user_delete {username: user_1}
```

## user_info

This job returns information about a given user (identified by username).

**Permissions**: Users: root | Groups: root, exaadm

**Parameters**:

- `username` (str, required): 'User name. Example: root.'
- `userid` (int, optional): 'User ID in integer format. Example: 1000.'

**Examples**:

```bash
confd_client user_info {username: root}
```

## user_list

This job returns a list of all users including all user details.

**Permissions**: Users: root | Groups: root, exaadm

## user_modify

This job modifies a given user (identified by username) and syncs the
changes on all nodes.

**Permissions**: Users: root | Groups: root, exaadm

**Parameters**:

- `username` (str, required): String containing the name of the user.
- `additional_groups` (list, optional): 'Comma-separated list of group names of additional groups that the user should be a member of. Example: [root, exausers].'
- `authorized_keys` (list, optional): A list of authorized keys for the user.
- `group` (str|int, optional): String containing the primary group ID or group name for the user.
- `login_enabled` (bool, optional): Boolean value defining if login is allowed (true) or not (false).

**Examples**:

```bash
confd_client user_modify {username: user_1}
```

## user_passwd

This job changes the password for a given user (identified by username) and
syncs the changes on all nodes.

**Permissions**: Users: root, xmlrpc user | Groups: root, exaadm

**Parameters**:

- `password` (str, required): String containing the new password for the user.
- `username` (str, required): String containing the name of the user.
- `encode_passwd` (bool, optional): Boolean value defining if the password is to be encoded (true) or not encoded (false).

**Examples**:

```bash
confd_client user_passwd {password: new_password, username: user_1}
```

## group_create

This job creates a new group with a given name and syncs the change on all
nodes.

**Permissions**: Users: root | Groups: root, exaadm

**Parameters**:

- `groupid` (int, required): Integer value containing the ID of the new group.
- `groupname` (str, required): String containing the name of the new group.

**Examples**:

```bash
confd_client group_create {groupid: 42, groupname: exausers}
```

## group_delete

This job deletes a group (identified by groupname) and syncs the change on
all nodes.

**Permissions**: Users: root | Groups: root, exaadm

**Parameters**:

- `groupname` (str, required): String containing the name of the group to delete.

**Examples**:

```bash
confd_client group_delete {groupname: exausers}
```

## group_info

This job returns details about a given group (identified by groupname).

**Permissions**: Users: root | Groups: root, exaadm

**Parameters**:

- `groupname` (str, required): String containing the group name.

**Examples**:

```bash
confd_client group_info {groupname: exausers}
```

## group_list

This job returns a list of all groups in the configuration. The job has no
parameters.

**Permissions**: Users: root | Groups: root, exaadm
