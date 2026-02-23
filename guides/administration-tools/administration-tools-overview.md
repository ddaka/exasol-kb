---
tool_name: confd_client
doc_type: concept
category: administration
technical_entities:
  - Exasol Admin
  - c4
  - ConfD
  - confd_client
  - EXAsupport
  - Administration API
summary: >
  Overview of Exasol administration tools — Exasol Admin web UI, c4 deployment
  tool, ConfD low-level API, EXAsupport log collection tool, and the
  Administration API (RESTful API built on ConfD).
---

# Administration Tools

This section describes the system administration interfaces and tools that are available in Exasol.

- **Exasol Admin** — Exasol Admin is a web-based user interface that makes it easy to carry out common administration tasks such as monitoring your database, generating and analyzing logs, creating backup schedules, updating Exasol, and managing files in BucketFS.

- **Exasol Deployment Tool (c4)** — c4 is a command line tool used to create and manage Exasol deployments. Through c4 you can also connect to the nodes in a cluster and run the administration tools confd_client and EXAsupport.

- **ConfD** — ConfD is a low-level API that enables you to carry out administrative tasks on an Exasol cluster using either the command-line tool confd_client or through XML-RPC in your own Python programs.

- **EXAsupport** — EXAsupport is a command-line tool used to create logs for troubleshooting. EXAsupport is available on all nodes in a cluster and is accessed through c4.

- **Administration API** — Exasol Administration API is a RESTful API built on top of ConfD, which allows you to carry out common administration tasks using RESTful commands through various programming languages and interfaces.
