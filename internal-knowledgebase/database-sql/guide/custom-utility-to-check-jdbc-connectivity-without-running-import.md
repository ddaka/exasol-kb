---
tool_name: internal-knowledgebase
doc_type: guide
category: database-sql
title: "Custom utility to check JDBC connectivity without running IMPORT"
summary: "Use a standalone Java test utility on cluster nodes to validate third-party JDBC connectivity outside IMPORT workflows."
---
# Custom utility to check JDBC connectivity without running IMPORT

## Purpose

When `IMPORT FROM JDBC` hangs or fails without clear cause, this utility helps prove whether the issue is in third-party JDBC connectivity (driver/remote DB/network path) rather than Exasol loader flow.

The utility runs in COS with the same Java runtime and JDBC JARs used by Loader.

## Prerequisites

- Uploaded JDBC driver in the cluster.
- Access to a COS node.
- `JdbcConnectionTest` utility artifacts (`JdbcConnectionTest.java` or compiled classes).
- A valid SQL query for the target external database.

## Configuration file

Create `JdbcConnectionTest.properties`:

```text
JDBC_CONNECTION_STRING = <jdbc-url>
DRIVER_CLASS = <driver-main-class>
JAR_PATH = <path-to-primary-jar>
DB_USER_NAME = <username>
SQL_STMT = <validation-sql>
```

## Exasol 7 workflow

1. Upload the JDBC driver via EXAoperation.
1. Locate driver files under a path similar to:

```text
/usr/opt/EXASuite-7/EXAClusterOS-<version>/var/jdbc/jdbc<id>/
```

1. Create a working folder on a node and place utility files there.
1. Build classpath (`JAR_LIST`) from all JDBC JARs in that driver directory.
1. Run:

```shell
java -cp JAR_LIST:. JdbcConnectionTest
```

## Exasol 8 workflow

1. Upload the JDBC driver via ConfD/driver management.
1. Locate driver files under a path similar to:

```text
/exa/data/bucketfs/bfsdefault/.dest/default/drivers/jdbc/<driver-folder>/
```

1. Create a working folder on a node and place utility files there.
1. Build classpath (`JAR_LIST`) from all JDBC JARs in that folder.
1. Run:

```shell
java -cp JAR_LIST:. JdbcConnectionTest.java
```

## Interpreting results

- Successful output confirms JDBC connection and query execution for the provided credentials and SQL.
- Authentication/network/driver issues typically produce explicit Java SQL exceptions.
- Use this output as objective evidence during vendor-side troubleshooting.

## Optional advanced tests

### JVM parameter checks

You can append JVM parameters to simulate Loader runtime tuning scenarios:

```shell
java -cp JAR_LIST:. JdbcConnectionTest -Xmx1024M
```

### Security manager policy checks

For file-access-denied style failures, test with explicit security policy:

```shell
java -cp JAR_LIST:. JdbcConnectionTest \
  -Djava.security.manager \
  -Djava.security.policy=<absolute-path-to-etljdbc.policy>
```

## Notes

- Use Java 8-compatible build/runtime when required by the target environment.
- Include all driver JARs in classpath for multi-JAR drivers.
- Keep credentials out of shell history where possible.

## References

- Exasol docs: Add JDBC driver (v7/v8) and Load Data from External Sources.
- Internal attachment repository for `JdbcConnectionTest` utility files.


