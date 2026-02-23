---
tool_name: confd_client
doc_type: guide
category: support
technical_entities:
  - log files
  - support
  - EXAsupport
  - Exasol Admin
  - profiling
  - backtrace
  - coredumps
summary: >
  Overview of log files for Exasol support cases — required information,
  debugging-specific use case requirements, and links to detailed log retrieval
  procedures.
---

# Log Files for Support

This section describes how to get system information that may be required when
you contact Support.

If you have an issue with your Exasol system and need help, create a support
case. To be able to troubleshoot and resolve your issue, our Support team will
require logs and other information about your system.

In some scenarios you may be asked to provide profiling information for a
problematic query. To get this information you must first run a test of the
problematic statement with profiling enabled, and then retrieve the log files for
that session.

In Exasol **2025.1+** you can use Exasol Admin to create an archive with the
necessary logs. For earlier versions, you must generate the logs manually using
EXAsupport.

## Required Information

| Field                          | Details                                               |
|--------------------------------|-------------------------------------------------------|
| What is the problem?           | Describe the issue as clearly as possible              |
| When did the problem occur?    | Narrow to a specific session or time of day            |
| Other relevant information     | Circumstances when the issue occurred                  |
| Log files and system info      | Provide relevant logs (see below)                      |

## Additional Information

The following details help accelerate analysis:

- Exasol version in use
- Number of data nodes
- Number of storage disks per node
- IP addresses of all Exasol nodes
- Network info (IP subnet, CIDR block, netmask)
- Backup strategy and location (local/remote)
- Time required for a full backup
- System configurations and modifications
- Database statistics

## Debugging Specific Use Cases

| Use Case                     | Required Logs                                         |
|------------------------------|-------------------------------------------------------|
| SQL/performance issues       | SQL + server process logs (before/after), profiling, session IDs |
| Aborted SQL processes        | SQL + server process logs, coredumps                  |
| Transaction conflicts        | SQL + server process logs for the sessions            |
| IMPORT/EXPORT issues         | SQL + server process logs for the entire day          |
| Database crash               | SQL + server process logs, ClusterOS logs, coredumps  |

## How to Get Logs and System Info

See the following guides:

- Generate Support Archive (Exasol Admin 2025.1+)
- Get All Logs
- Cluster Logs
- Logs for Server Processes
- Logs for SQL Processes
- Logs for SQL and Server Processes
- Backtrace Information
- Profiling Information
