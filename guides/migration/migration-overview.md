---
tool_name: confd_client
doc_type: guide
category: migration
technical_entities:
  - migration
  - Exasol 7.1
  - Exasol 8
  - backup
  - restore
  - maintenance window
summary: >
  Migration overview from Exasol 7.1 to Exasol 8 — explains why there is no
  direct update path, the two migration scenarios (new hosts vs. existing
  hardware), maintenance window estimation, and frequently asked questions.
---

# Migrate from Exasol 7.1

This section explains how to migrate an Exasol 7.1 database to the newest version of Exasol.

---

## Introduction

Because of the differences in architecture that was introduced with Exasol 8 there is no direct update path from Exasol 7.1 to later versions. Instead, you must create a new Exasol deployment and restore a full backup of your existing Exasol 7.1 database on the new (empty) database.

The migration path is different depending on whether you are planning to create the new deployment on your existing hardware, or if you are deploying on new hardware or virtual instances in the cloud. Deploying on existing hardware will require a longer maintenance window, since you need to take your database offline while installing an operating system and Exasol software. You will also have to reinstall Exasol 7.1 and restore from backup in case the migration fails. If you are migrating to new hardware, you can simply put the old cluster online again to resume operations while you troubleshoot.

This section describes the procedures for both of these migration scenarios:

- **Migrate to deployment on new hosts** — shorter maintenance window, old cluster remains available for fallback.
- **Migrate to deployment on existing hardware** — longer maintenance window, existing database must be taken fully offline.

To avoid excessive downtime and prevent a risk of accidental data loss, read through the instructions carefully before you start the migration procedure.

If your database is running Exasol 7.0, you must update to Exasol 7.1 before proceeding.

---

## Maintenance Window

The process of migrating your database to Exasol includes a planned maintenance window when the database is not available to users. The downtime begins at the step when you restart the database on a different port and ends when the backup has been successfully restored in Exasol.

The total required downtime is mainly determined by the restore time, since the majority of the backup time will be outside of the maintenance window. A restore operation will usually take approximately the same amount of time as the backup.

Tables with partitioning will have zone maps enabled by default when they are restored in the Exasol 8 database. This can take some time depending on the contents of the table.

### Example

| Operation | Elapsed Time | Downtime |
|-----------|-------------|----------|
| Create level 0 backup | 5 hours | 0 |
| Create level 1 backup | 30 minutes | 0 |
| Create level 2 backup | 30 minutes | 30 minutes |
| Restore full backup (level 0 + 1 + 2) | 5 hours | 5 hours |
| **Total downtime** | | **5 hours 30 minutes** |

If you are reusing the existing hardware, the time needed to install the OS on the hosts and to install Exasol must also be added to the planned downtime.

---

## FAQ

- **What versions of Exasol are available?** — Exasol can be installed as a Linux application on hardware or on virtual instances in AWS, Azure, and GCP. You can also deploy Exasol as a native cloud application on AWS.

- **Can I use my existing license for Exasol?** — No. Exasol 7.1 licenses are not valid in Exasol 8 or later because of differences in the license format. Contact Support for a new license.

- **Will Exasol still manage the OS patches?** — No. Exasol is provided as a software package that you install on a cloud service or on your own hardware, using a Linux distribution of your choice. Exasol services do not cover the management of the operating system.

- **Where can I get the change log?** — The release notes for all currently supported Exasol database releases are found in the Release Notes section. Each release note only describes the changes for that specific release.

- **How long will the maintenance window be?** — The downtime will depend on the restore time and, if necessary, the software installation time. See the Maintenance Window section above.

- **Does Exasol 8 come with an administration application like EXAoperation?** — Exasol Admin is a new web-based administration interface available in Exasol 2025.1 and later. Exasol 8 also includes several powerful APIs and configuration/installation tools.

- **Is a Management Server (License Server) still required?** — No. Exasol does not use a separate server for cluster management. The administrative interfaces are available on all nodes.

- **Does Exasol have a monitoring service?** — Yes. See the Exasol Monitoring Service FAQ for details.
