---
tool_name: internal-knowledgebase
doc_type: guide
category: database-sql
title: "Use exajload for Fast File Import"
summary: "Quick-start guide for using exajload (bundled with Exasol JDBC packages) to execute high-speed IMPORT FROM LOCAL CSV commands from client hosts."
---

# Use exajload for Fast File Import

## Overview

`exajload` is a command-line helper bundled with Exasol JDBC client packages for fast data loading from local files using `IMPORT FROM LOCAL CSV`.

## Prerequisites

- Exasol JDBC package installed (Windows or Linux).
- `exajload` available in the installation directory or `PATH`.
- Reachable Exasol database endpoint and valid credentials.
- Input file accessible from the client host where `exajload` runs.

## Procedure

### 1. Verify installation

Windows example:

```bat
"C:\Program Files (x86)\EXASOL\EXASolution-<version>\JDBC\exajload.exe" --help
```

Linux example:

```bash
exajload --help
```

### 2. Run a basic import

Windows example:

```bat
"C:\Program Files (x86)\EXASOL\EXASolution-<version>\JDBC\exajload.exe" ^
  -c 192.168.1.1:8563 ^
  -u sys ^
  -P "<password>" ^
  -sql "IMPORT INTO myschema.bigtable FROM LOCAL CSV FILE 'C:\\bigfile.txt'"
```

Linux example:

```bash
exajload \
  -c 192.168.1.1:8563 \
  -u sys \
  -P '<password>' \
  -sql "IMPORT INTO myschema.bigtable FROM LOCAL CSV FILE '/data/bigfile.txt'"
```

## Validation

- Check `exajload` exit code.
- Validate imported row counts in target table.
- Review rejected/error rows if import options define them.

## Notes

- Tune `IMPORT` format options (delimiters, encoding, null handling) to match source data.
- Keep credentials out of command history where possible.

## References

- [IMPORT Documentation](https://docs.exasol.com/sql/import.htm)
- [JDBC Driver Documentation](https://docs.exasol.com/connect_exasol/drivers/jdbc.htm)
- <https://www.youtube.com/watch?v=yjPt5o9CPU0>
