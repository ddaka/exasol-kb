---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: database-sql
title: "Spark Write Failures to Exasol via JDBC"
summary: "Troubleshooting guide for Spark JDBC write errors to Exasol, including CREATE TABLE type mismatches and CLOB/null handling issues."
---

# Spark Write Failures to Exasol via JDBC

## Context

Generic Spark JDBC support can be used with Exasol when the dedicated Spark connector is not feasible.

- Spark JDBC docs: <https://spark.apache.org/docs/latest/sql-data-sources-jdbc.html>
- Preferred connector (when compatible): <https://github.com/exasol/spark-exasol-connector>

## Symptoms

Common failures when writing with `overwrite` or `append` modes:

1. Spark-generated `CREATE TABLE` is incompatible with Exasol.
2. JDBC write fails with datatype errors such as `Cannot handle data type: 2005`.

## Root Causes

1. Spark may map string-like columns to JDBC types that resolve to `TEXT`, which Exasol does not accept in this context.
2. Null-containing dataframes may produce CLOB-style transfers for some fields, causing unsupported type handling errors.

## Resolutions

### 1. Control target column definitions

Use one of the following patterns:

- Provide explicit column types via `createTableColumnTypes` in `overwrite` mode.
- Pre-create the table in Exasol and write with `append` mode.

### 2. Normalize null values before write

Replace null values where appropriate before persistence:

```scala
myDF.na.fill("")
```

Use type-appropriate fills for non-string columns.

## Working Examples

Start Spark with Exasol JDBC driver:

```bash
./bin/spark-shell --driver-class-path path/to/exajdbc.jar --jars exajdbc.jar
```

Write with explicit type mapping (`overwrite`):

```scala
myDF.na.fill("").write
  .mode("overwrite")
  .format("jdbc")
  .option("url", "jdbc:exa:192.168.56.1..3:8563")
  .option("user", "sys")
  .option("password", "<password>")
  .option("dbtable", "myschema.mytable")
  .option("createTableColumnTypes", "mycolumn VARCHAR(10)")
  .save()
```

Write into existing table (`append`):

```scala
myDF.na.fill("").write
  .mode("append")
  .format("jdbc")
  .option("url", "jdbc:exa:192.168.56.1..3:8563")
  .option("user", "sys")
  .option("password", "<password>")
  .option("dbtable", "myschema.mytable")
  .save()
```

## Validation

- Verify target schema/table datatypes in Exasol before writes.
- Test with a small dataframe containing representative null patterns.
- Confirm Spark job completes without JDBC type-conversion errors.
