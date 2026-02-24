---
tool_name: internal-knowledgebase
doc_type: guide
category: database-sql
title: "Use Oracle BI Enterprise Edition (OBIEE) with Exasol"
summary: "Configure ODBC connectivity, adjust OBIEE datasource capabilities, and apply SQL preprocessing for OBIEE compatibility with Exasol."
---

# Use Oracle BI Enterprise Edition (OBIEE) with Exasol

## Overview

This guide describes the core setup required to connect OBIEE to Exasol using ODBC.

## Prerequisites

- Exasol ODBC driver installed on OBIEE server.
- Exasol ODBC driver installed on OBIEE Administrator Windows client.
- Matching DSN name on both server and client.

## Procedure

### 1. Configure ODBC connection

OBIEE includes DataDirect drivers. Typical library path:

- `$ORACLE_HOME/common/ODBC/Merant/7.1.4/lib/`

When running `config_odbc.sh`, ensure the DataDirect library path is included in runtime search paths.

`config_odbc.sh` generates DSN entries in `~/odbc.ini`; copy the Exasol DSN into OBIEE's `odbc.ini`.

Typical OBIEE `odbc.ini` location:

- `ORACLE_INSTANCE/bifoundation/OracleBIApplication/coreapplication/setup/odbc.ini`

After DSN changes, restart OBIEE services (at minimum Presentation Services).

### 2. Align OBIEE data source capabilities

Update OBIEE `DBFeatures.INI` so Exasol uses compatible settings (section `DATA_SOURCE_FEATURE = ODBC_300`) from the provided template.

Typical config location:

- `ORACLE_INSTANCE/config/OracleBIServerComponent/coreapplication_obisn`

In OBIEE Administrator:

- Data source type: `ODBC Advanced`
- Connection pool -> Call Interface: `ODBC 300`

### 3. Install Exasol SQL preprocessor for OBIEE compatibility

Apply the preprocessor SQL from the provided script package.

Typical compatibility adjustments include:

- Type translation (`TEXT` -> `VARCHAR`)
- ODBC function translation (`DATEADD`, `TIMESTAMPADD`, `DATEDIFF`, `TIMESTAMPDIFF`, and related date functions)
- Escaping fixes for date formatting expressions

Activation example:

```sql
ALTER SYSTEM SET sql_preprocessor_script = dbguru.preprocess_ODBC();
```

## Validation

- OBIEE can open Exasol connection pool successfully.
- Test analyses run without function/type translation errors.
- Generated SQL executes in Exasol without parser failures.

## Downloads

- [preprocess_OBIEE_ODBC_v4.sql](https://github.com/exasol/internal-knowledgebase/blob/main/Connect-with-Exasol/attachments/preprocess_OBIEE_ODBC_v4.sql)
- [OBIEE_DBFeatures_EXASOL.INI](https://github.com/exasol/internal-knowledgebase/blob/main/Connect-with-Exasol/attachments/OBIEE_DBFeatures_EXASOL.INI)

## Reference

- <http://docs.oracle.com/cd/E17904_01/bi.1111/e10540/deploy_rpd.htm#BIEMG1177>
