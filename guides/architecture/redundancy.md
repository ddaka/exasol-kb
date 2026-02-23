---
tool_name: confd_client
doc_type: concept
category: architecture
technical_entities:
  - redundancy
  - storage volumes
  - data replication
  - failover
  - reserve nodes
  - mirror
summary: >
  Redundancy in Exasol — explanation of the redundancy attribute on storage
  volumes, how data is mirrored across active cluster nodes, and failover
  behavior when a node fails.
---

# Redundancy

This article explains the redundancy attribute on storage volumes in Exasol.

Redundancy is an attribute that you set when you create a storage volume. The redundancy level specifies the number of copies of the data that is hosted on active cluster nodes.

- **Redundancy 1** = no redundancy. If a node fails, the data volume is no longer available. Redundancy 1 is typically only seen with single node systems.
- **Redundancy 2** = each node holds a copy of the data that is operated on by the neighbor node. If a node fails, the data volume remains available. Redundancy 2 is best practice in multiple-node systems and recommended by Exasol.

For information about how to set redundancy when creating a volume, see **Create Data Volume** and **Create Local Archive Volume**.

---

## Example

In this example, the volumes are configured with redundancy 2. When node n11 modifies A, the mirror A' on the neighbor node n12 is synchronized over the private network. If node n11 fails, the reserve node n15 starts up and data is copied to it from the mirror on n12.

For more information about the failover mechanisms, see **Fail safety**.
