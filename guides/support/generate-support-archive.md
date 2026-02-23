---
tool_name: confd_client
doc_type: guide
category: support
technical_entities:
  - support archive
  - Exasol Admin
  - logs
summary: >
  How to generate a support archive with logs and system information using
  Exasol Admin — prerequisites, source selection (cluster logs, backtraces,
  database logs), and download procedure.
---

# Generate Support Archive in Exasol Admin

In on-premises deployments of Exasol **2025.1+** you can use Exasol Admin to
create an archive with logs and other information to include with your support
case. For earlier versions you must use EXAsupport.

## Prerequisites

- Database version must be Exasol 2025.1 or later
- Exasol Admin must be enabled
- Database must be running

## Procedure

1. Log in to Exasol Admin.
2. On the **Databases** page, click **Support archive**.
3. On the Support archive page, use the dropdown menus to select the logs and
   backtraces to retrieve, and the databases and nodes to collect from. You must
   select at least one node and at least one of:
   - Cluster logs
   - Backtraces
   - Database logs
   - Databases
4. To collect information about a specific SQL session, enter the session ID in
   **Sessions**. To get the session ID, query `EXA_ALL_SESSIONS` or
   `EXA_USER_SESSIONS`.
5. Click **Create and download support archive**.

Exasol Admin generates the specified logs and creates an archive in tarball
format (`*.tar.gz`) that you can download.
