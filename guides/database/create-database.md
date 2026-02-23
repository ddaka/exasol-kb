---
tool_name: confd_client
doc_type: guide
category: database-management
subcommands:
  - db_create
  - db_list
technical_entities:
  - database
  - data volume
  - password hash
summary: >
  How to create an Exasol database — db_create parameters, password hash
  generation with c4 pwdhash, and verification with db_list.
---

# Create a Database

A database is created automatically during deployment. Create additional
databases only for testing, maintenance, or after deleting the existing database.

## Prerequisites

- No database with the same name or UUID must already exist

## Procedure

1. Connect to COS:

```bash
c4 connect -i <PLAY_ID> -s cos
```

2. Create the database:

```bash
confd_client db_create \
  db_name: MY_DEV_DATABASE \
  version: 8.32.0 \
  data_volume_name: DATA_VOLUME_1 \
  mem_size: '2048 MiB' \
  nodes: '[11, 12, 13, 14, 15]' \
  num_active_nodes: 4 \
  default_sys_passwd_hash: '9B37963C9904E08B4516711AAA5840206B3817F9B671C385F00BFC2E'
```

| Parameter                | Type    | Description                                                    |
|--------------------------|---------|----------------------------------------------------------------|
| `db_name`                | string  | Unique database name                                           |
| `version`                | string  | Database version (e.g. `8.32.0`)                               |
| `data_volume_name`       | string  | Existing data volume name                                      |
| `mem_size`               | string  | RAM allocation (e.g. `2048 MiB`, `4 GiB`)                     |
| `nodes`                  | list    | Node IDs (active + reserve)                                    |
| `num_active_nodes`       | integer | Number of active nodes (must match data volume master nodes)   |
| `default_sys_passwd_hash`| string  | Password hash for SYS user                                     |

### Generate Password Hash

```bash
c4 pwdhash -i
```

## Verification

```bash
confd_client db_list
```
