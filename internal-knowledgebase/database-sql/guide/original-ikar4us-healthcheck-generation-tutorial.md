---
tool_name: internal-knowledgebase
doc_type: guide
category: database-sql
title: "Original Ikar4us healthcheck generation tutorial"
summary: "Generate offline healthcheck PowerPoint reports from sol-520 statistical exports using the legacy Ikar4us tool."
---
# Original Ikar4us healthcheck generation tutorial

## Purpose

For customers not connected to online monitoring, use the legacy Ikar4us Java tool to build healthcheck reports from `sol-520` statistical exports.

## Prerequisites

- `sol-520` statistical export from customer database.
- Java runtime available on workstation.
- Local Exasol sandbox database for data import/processing.

## Step 1: Prepare local workspace

1. Unpack Ikar4us package and place `IKAR4US_ROOT` locally.
1. Copy template folder for your run (for example `Report1`).
1. Extract `sol-520` files into:

```text
IKAR4US_ROOT\Report1\db\Sol520
```

1. Ensure filenames match expected export script output.

## Step 2: Configure application

### File: `IKAR4US_ROOT\Report1\db\IKAR4us.config`

Set:

- DB connection details.
- Java executable path.
- EXAplus path.
- Analysis/config paths.

Example:

```properties
app.CONNECTIONSTRING=jdbc:exa:<DB_HOST>/<FINGERPRINT>:<PORT>
app.CONNECTIONSTRING_EXAPLUS=jdbc:exa:<DB_HOST>:<PORT>
app.USER=sys
app.PASSWORD=exasol
app.JAVA=C:\\Program Files\\Java\\jdk-18.0.2.1\\bin\\java.exe
app.EXAPLUS=C:\\Program Files\\Exasol\\EXASolution-7.1\\EXAplus\\exaplus.jar
app.ANALYSIS_PATH=C:\\Users\\<user>\\IKAR4US_ROOT\\Report1\\db
app.config_path=C:\\Users\\<user>\\IKAR4US_ROOT\\Report1\\db
```

Note: intentionally invalid `app.config_path` can force reimport/rebuild, but use carefully.

### File: `IKAR4US_ROOT\Report1\db\Sol520\1.config`

Set report metadata such as customer name, DB name, reporting period, and aggregation settings.

## Step 3: Run Ikar4us

```shell
java.exe -jar com.exasol.ikar4us-0.0.1-SNAPSHOT-jar-with-dependenciesV4.jar C:\Users\<user>\IKAR4US_ROOT\Report1\db
```

## Step 4: Validate output

Expected artifact:

```text
IKAR4US_ROOT\Report1\db\HealthCheck.pptx
```

Verify charts are populated (no empty slides/graphs).

## Known issue: Exasol v8 exports

The legacy tool was built against older statistics structures and may fail on v8 datasets.

Workaround pattern:

1. Run tool once and capture failing objects.
1. In sandbox DB, repair affected `USER_PROFILE` tables.
1. Use internal `v8_import_fix.sql` if applicable.
1. Re-run without triggering full forced reimport.

## References

- `sol-520` export article: <https://exasol.my.site.com/s/article/Statistics-export-for-support?language=en_US>
- Ikar4us template package (internal link): SharePoint source used by support team.
