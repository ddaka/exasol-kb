---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: database-sql
title: "EXA IMPORT/EXPORT and EXAmigration IP Mapping Requirement"
summary: "Troubleshooting guide for ETL-4212 and related EXA-to-EXA transfer failures caused by invalid node-to-external-node IP mapping patterns."
---

# EXA IMPORT/EXPORT and EXAmigration IP Mapping Requirement

## Problem

`IMPORT FROM EXA` and `EXPORT INTO EXA` can fail when node IP mapping between internal and external node numbers is inconsistent.

## Requirement

Node number and external node number assignments must follow a consistent offset (equal distance) across all participating nodes.

Example (valid offset `+10`):

| Node Number | External Node Number |
| --- | --- |
| 11 | 21 |
| 17 | 27 |
| 18 | 28 |
| 19 | 29 |

Example (invalid mapping):

| Node Number | External Node Number |
| --- | --- |
| 11 | 21 |
| 17 | 37 |
| 18 | 28 |
| 19 | 26 |

## Symptoms

Typical errors include:

```text
ETL-4212: Parallel connection from <source host> to external EXASolution at <destination host and port> failed. [No server listening.]
```

```text
[Code: 0, SQL State: ETL-4] Connection from <source host> to external EXASolution at <destination host and port> failed. [Connection attempt timed out No server listening.]
```

## Workaround

Use `IMPORT FROM JDBC` / `EXPORT INTO JDBC` with an explicit connection string to the target Exasol instance.

Important trade-off:

- JDBC transfer is not parallel in the same way as EXA-to-EXA transport and can be significantly slower for large data volumes.

## Notes

Historical tracking reference:

- <https://exasol.atlassian.net/browse/SPOT-16272>
