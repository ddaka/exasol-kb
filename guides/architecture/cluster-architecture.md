---
tool_name: confd_client
doc_type: concept
category: architecture
technical_entities:
  - cluster
  - nodes
  - reserve nodes
  - shared-nothing
  - MPP
  - SPMD
  - EXAStorage
  - BucketFS
  - data volumes
  - backup volumes
  - private network
  - public network
summary: >
  Cluster architecture overview — database layer (shared-nothing MPP, active
  and reserve nodes), storage layer (EXAStorage data/backup volumes, BucketFS),
  network layer (private and public networks), and software components.
---

# Cluster Architecture

This article provides an overview of the architecture of an Exasol cluster.

---

## Database Layer

An Exasol database runs in a cluster that contains one or more nodes. Each node is a host machine that has its own CPUs and main memory (RAM). The total number of CPUs and the total amount of RAM in a cluster determines the compute power.

A cluster can have any number of reserve nodes on standby in addition to the active data nodes. A reserve node is a node that has the same basic configuration as a data node, but which is not active and contains no data. If the nodes are set up with redundancy > 1, an active node holds a mirror of the data on the neighboring node. If one of the active nodes fails, a reserve node will automatically take over the role of the failed node. Data is then copied to this node from a mirror on one of the other active nodes.

For more details about setting up redundancy in nodes, see **Redundancy**.

For information about how to manage clusters and nodes, see **Nodes Management**.

### Shared-Nothing Architecture

Exasol is a massively parallel processing (MPP) database designed on a shared-nothing architecture (SN). This means that data is distributed across all nodes in the cluster and constantly synchronized between the nodes. All nodes operate on an equal basis, there is no master node. This processing paradigm is also known as SPMD (single program multiple data).

When a query is sent to the database, the query is first accepted by the node that the client is connected to and then distributed to all nodes in the cluster over the private network. Algorithms optimize the query, determine the best plan of action, and generate indexes as needed. The database then processes the partial results based on the local data sets and delivers the global results back to the user through the original connection.

---

## Storage Layer

Data storage in Exasol is managed by a distributed storage engine, EXAStorage. Data is stored in data volumes that can be located on either local or remote storage services. Database backups are stored on separate backup volumes, which can also be created either in the cluster or on remote storage.

### BucketFS

BucketFS is a synchronous file system that is available on all database nodes in an Exasol cluster, and which is used for storing files such as drivers and scripts for user-defined functions (UDF scripts). Each node in the cluster can connect to the BucketFS service and will see the same content as the other nodes. The data stored in BucketFS is replicated locally on each node and automatically synchronized.

All configured data disks in Exasol have a preinstalled BucketFS service with a default bucket. You can create additional BucketFS services and buckets as needed.

The data in BucketFS is not part of the database backups and must be backed up separately.

---

## Network Layer

An Exasol cluster is normally configured with two networks using separate physical interfaces: a private network for internal communication between the nodes in the cluster, and a public network that allows connections to the database from outside of the cluster.

In Exasol, the physical networks must be configured using the network management interfaces on the Linux hosts. You can then specify the private and public IP addresses for the nodes in the deployment configuration.

---

## Software

Exasol provides several administration interfaces that enable you to deploy and manage your databases and clusters using both built-in and external tools. The administration interfaces are available on all nodes in a cluster.

For more information about the administration interfaces, see **Administration Tools**.
