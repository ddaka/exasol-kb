---
tool_name: confd_client
doc_type: guide
category: database-management
subcommands:
  - db_stop
  - db_state
technical_entities:
  - database
  - shutdown
  - forced shutdown
  - Exasol Admin
summary: >
  How to stop an Exasol database — normal vs forced shutdown behaviour,
  Exasol Admin and confd_client db_stop procedures, and verification.
---

# Stop a Database

## What Happens During Shutdown

- `SHUTDOWN` event written to `EXA_SYSTEM_EVENTS`
- All sessions killed (including running queries)
- Audit entries written to `EXA_DBA_AUDIT_SQL`, `EXA_DBA_AUDIT_SESSIONS`

## Exasol Admin

1. Open Databases page
2. Click the **Stop database** button
3. Status changes from Running to Stopped

## Command Line

```bash
c4 connect -i <PLAY_ID> -s cos
confd_client db_stop db_name: MY_DATABASE
```

| Parameter  | Type    | Description                                              |
|------------|---------|----------------------------------------------------------|
| `force`    | boolean | Force shutdown (use only when normal stop fails)         |

## Forced vs Normal Shutdown

| Aspect              | Normal Shutdown                               | Forced Shutdown                              |
|---------------------|-----------------------------------------------|----------------------------------------------|
| Goal                | Graceful termination with audit trail         | Immediate termination                        |
| Usage               | Standard procedure                            | Last resort when normal shutdown is blocked  |
| Duration            | Variable (up to several minutes)              | Immediate                                    |
| Active transactions | Aborted and rolled back                       | Abruptly terminated, no rollback recorded    |
| Auditing            | Statistics flushed and written                | No auditing entry written                    |

## Verification

```bash
confd_client db_state db_name: MY_DATABASE
```

Result `setup` confirms the database is stopped.
