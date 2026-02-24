---
tool_name: cos
doc_type: guide
category: system
title: "INTERNAL: technical overview of licensing"
summary: "Operational guidance for interpreting Exasol license behavior in support cases and correlating it with database settings."
---
# INTERNAL: technical overview of licensing

## Purpose

Support cases often require understanding license scope, where license state is visible, and how DB-level limits relate to cluster-level license enforcement.

## Key points

1. Current license models:
- Raw data license
- Database RAM license

2. Legacy compressed-data contracts can still exist for older customers. Contract terms may differ from currently sold models.

3. `EXA_SYSTEM_EVENTS` includes:

| COLUMN_NAME | COLUMN_COMMENT |
| --- | --- |
| DB_RAM_SIZE | Used DB RAM license in GiB |

4. Database parameters `rawSizeLimitInMiB` and `memSizeLimitInMiB` are DB-level controls, not the customer contract itself.

5. License enforcement scope is cluster-wide. DB-level limits are used to distribute capacity between databases.

6. Operational behavior observed by Ops: if configured DB RAM exceeds license allowance, database start can fail.

## Support interpretation guide

- Treat uploaded license content (XML) as the source of contractual limits.
- Treat DB-level size/RAM settings as operational partitioning controls.
- Avoid promoting `rawSizeLimitInMiB` as a standard recommendation unless explicitly required by a known use case.

## Canonical references (de-duplication)

- `documents/cos/confd-system-and-infrastructure.md` (`license_upload`, `license_info`, `license_run_check`)
- `documents/cos/cos_directory0structure.md` (license file locations)
- `documents/c4/c4_troubleshooting.md` (license issue triage context)

## External and internal references

- [Licenses](https://docs.exasol.com/planning/licensing.htm)
- [How to configure database raw and memory size limits](https://exasol.my.site.com/s/article/How-to-configure-database-raw-and-memory-size-limits)
- [License size limit enforcement by dwad](https://exasol.atlassian.net/wiki/spaces/RD/pages/12160962/License+size+limit+enforcement+by+dwad)
