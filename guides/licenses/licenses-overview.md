---
tool_name: confd_client
doc_type: concept
category: licensing
subcommands:
  - license_info
  - license_run_check
technical_entities:
  - license
  - raw data size
  - restricted mode
  - license limits
summary: >
  Exasol licensing overview — raw-data-based license model, license limits,
  restricted mode behaviour (blocked statements and R0010 error), and
  recommended recovery actions when limits are exceeded.
---

# Licenses

This section describes how licensing works in Exasol.

---

## License Model

Exasol uses a raw-data-based license model. The raw data size corresponds to the amount of uncompressed data stored across all databases in the cluster, comparable to the size the data would have if stored as CSV files.

---

## License Limits

When a database exceeds its licensed raw data size, Exasol enters **restricted mode**. In restricted mode, the following SQL statements are blocked:

- `IMPORT`
- `INSERT`
- `CREATE TABLE AS`
- `MERGE`
- `SELECT INTO`

Attempts to execute these statements return error code **R0010**.

Audit data is not counted towards the raw data license limit.

---

## Restricted Mode Recovery

To recover from restricted mode:

1. Remove data using `DROP` or `DELETE` statements
2. Run `FLUSH STATISTICS` to update the raw data size
3. Check the current database size using the `EXA_DB_SIZE_LAST_DAY` system table

---

## Check License

To view license details and verify that databases do not exceed the license limits, see **Check License**.
