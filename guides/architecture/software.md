---
tool_name: confd_client
doc_type: concept
category: architecture
technical_entities:
  - COS
  - EXAStorage
  - Exasol Admin
  - ConfD
  - confd_client
  - c4
  - XML-RPC
summary: >
  Software overview for Exasol on-premises — cluster operating system (COS),
  EXAStorage distributed storage engine, Exasol Admin web UI, ConfD API,
  confd_client CLI, and c4 deployment tool.
---

# Software

This article provides an overview of the applications and interfaces in your Exasol system.

---

## Overview

An Exasol on-premises installation is deployed as an application in a Linux host operating system, which can be running on hardware or as cloud instances. Each node will contain the Exasol database engine, the cluster operating system (COS), the storage engine (EXAStorage), and software for system administration.

**Exasol cluster operating system (COS)** manages and monitors the nodes in the cluster. COS also manages communication between the nodes over the private (internal) network, and between the database and the database users and administrators over the public (external) network.

**EXAStorage** is a distributed storage engine that manages the data storage and the failover mechanisms in an Exasol deployment. Data is stored in data volumes, which can be located on either local or remote storage devices. Database backups are stored on separate backup volumes, which can be created either in the cluster or on remote storage.

**Exasol Admin** is an easy-to-use web interface that allows you to carry out common administration tasks such as monitoring your database, generating and analyzing logs, creating backup schedules, updating Exasol, and managing files in BucketFS.

**ConfD** is a low-level API that can be used to perform all administrative actions in Exasol. The ConfD API is available on all nodes in a deployment. You can interact with ConfD directly using the command-line tool confd_client, or indirectly through XML-RPC in your own Python programs.

**Exasol Deployment Tool (c4)** is a multi-purpose command line tool that is used to create, configure, and manage Exasol deployments on all supported platforms. You can also use c4 to access other administration interfaces in Exasol.

Since Exasol does not include a host operating system, you must prepare the host machines according to the system requirements for your installation platform before you deploy Exasol.

---

## Connect to Exasol

Once your Exasol database is up and running, you can connect to it using various compatible database clients and other tools.
