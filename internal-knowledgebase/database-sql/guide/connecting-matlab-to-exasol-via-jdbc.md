---
tool_name: internal-knowledgebase
doc_type: guide
category: database-sql
title: "Connect MATLAB to Exasol via JDBC"
summary: "Configure MATLAB Database Toolbox with the Exasol JDBC driver and run basic read/write operations against Exasol."
---

# Connect MATLAB to Exasol via JDBC

## Overview

This guide explains how to configure MATLAB to connect to Exasol using the JDBC driver.

## Prerequisites

- MATLAB with Database Toolbox installed.
- Exasol JDBC driver (`exajdbc.jar`) available on the local machine.
- Reachable Exasol endpoint and valid credentials.

Recommended MATLAB settings:

- Database Toolbox:
  - Null data handling: `null`
  - Data return format: `cellarray`
  - Error handling: `report`
  - Fetch in batches: enabled (`10000`)
- Database Explorer import batch size: `10000`

## Procedure

### 1. Add Exasol JDBC driver to MATLAB classpath

Option A: `classpath.txt`

1. Open `$MATLABROOT/toolbox/local/classpath.txt`.
2. Add the full path to `exajdbc.jar`.
3. Restart MATLAB.

Option B: `javaclasspath.txt`

1. Run `prefdir` in MATLAB.
2. In that directory, create `javaclasspath.txt`.
3. Add the full path to `exajdbc.jar`.
4. Restart MATLAB.
5. Run `javaclasspath` and verify the driver is listed under static classpath.

### 2. Initialize a JDBC connection

```matlab
schema = 'TEST';
user = 'SYS';
passw = '<password>';
driver = 'com.exasol.jdbc.EXADriver';
conn_str = 'jdbc:exa:<host_or_ip>:8563';

conn = database(schema, user, passw, driver, conn_str);
```

### 3. Read data from Exasol

```matlab
res = fetch(conn, 'SELECT * FROM TEST.CUSTOMERS');
```

### 4. Write data to Exasol

```matlab
datainsert(conn, 'TEST.MATLAB', {'n1','n2'}, res);
```

## Validation

- Connection object initializes without errors.
- `fetch` returns expected rows/columns.
- `datainsert` completes and inserted rows are queryable in Exasol.

## References

- <https://mathworks.com/products/database/>
- <https://mathworks.com/help/database/index.html>
