---
tool_name: confd_client
doc_type: guide
category: database-management
subcommands:
  - db_start
  - db_state
  - db_list
technical_entities:
  - database
  - Exasol Admin
summary: >
  How to start an Exasol database — via Exasol Admin UI or confd_client
  db_start, with verification using db_state.
---

# Start a Database

Available via **Exasol Admin** (2025.1+) or command line.

## Exasol Admin

1. Open Databases page
2. Click the **Start database** button
3. Status changes from Stopped to Running
4. Click **Test** to verify connectivity

## Command Line

```bash
c4 connect -i <PLAY_ID> -s cos
confd_client db_list
confd_client db_start db_name: MY_DATABASE
```

## Verification

```bash
confd_client db_state db_name: MY_DATABASE
```

Result `running` confirms the database is started.
