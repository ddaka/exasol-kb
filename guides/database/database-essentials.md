---
tool_name: confd_client
doc_type: concept
category: database-management
technical_entities:
  - database
  - cluster
  - nodes
  - reserve nodes
  - RAM
summary: >
  Database essentials — how an Exasol database runs in a cluster with active
  and reserve nodes, and the warning about running multiple databases on the
  same node.
---

# Database Essentials

An Exasol database runs in a cluster of one or more nodes. Each node has its own
CPUs and RAM — the total determines compute power.

A cluster can have **reserve nodes** on standby. With redundancy > 1, active
nodes hold mirrors of neighbouring node data. If an active node fails, a reserve
node takes over automatically and data is copied from a mirror.

> **Warning:** Do not run multiple databases on the same node in production.
> Each database assumes full control over node resources. Combined DB RAM must
> not exceed the recommended maximum for a single database.
