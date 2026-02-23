---
tool_name: confd_client
doc_type: guide
category: migration
technical_entities:
  - migration
  - existing hardware
  - backup
  - restore
  - remote archive volume
  - BucketFS
  - level 0 backup
  - level 1 backup
  - level 2 backup
  - bare metal
summary: >
  Step-by-step guide to migrate Exasol 7.1 to a new deployment on existing
  hardware — prerequisites, 9-step migration procedure (full backup, incremental
  backups, change port, shutdown, install OS, install Exasol, remote archive
  volume, restore), fallback procedure, and next steps.
---

# Migrate to Deployment on Existing Hardware

This article explains how to migrate an Exasol 7.1 database to a new deployment on existing hardware.

Migrating an Exasol 7.1 database to existing hardware will require a longer maintenance window compared to migrating to new hardware or virtual instances in the cloud, because the existing database must be taken offline before you install the new database.

This deployment option is best suited for installations of Exasol running on bare metal with modern hardware.

---

## Prerequisites

- The remote archive volume used for Exasol 7.1 backups must be accessible to the new deployment.
- The hosts must meet all system requirements for an Exasol installation.
- You must have a valid Exasol license in the new license format.

Exasol 7.1 licenses are not valid in later versions of Exasol because of differences in the license format. Contact Support for a new license.

---

## Migration Procedure

Files in BucketFS, such as UDF scripts and drivers, are not included in the database backup. These files must be manually backed up and then uploaded to the Exasol 8 database. We recommend using BucketFS Client to download and upload files in BucketFS.

### Step 1: Create a Full Backup of the Exasol 7.1 Database

In Exasol 7.1, create a level 0 (full) backup on a remote archive volume that can also be accessed by the new deployment.

### Step 2: Create a Level 1 Backup of the Exasol 7.1 Database

Run a level 1 incremental backup to save any changes that were committed during the level 0 (full) backup.

### Step 3: Restart the Exasol 7.1 Database on a Different Port

To make sure that clients cannot connect to the Exasol 7.1 database during the migration procedure, you must change the connection port of the database. This step requires that you temporarily stop the database.

> **The database downtime begins at this point.**

### Step 4: Create a Level 2 Backup of the Exasol 7.1 Database

After restarting the database, run a level 2 incremental backup to make sure that any changes to the database since the level 1 backup was started are also backed up.

### Step 5: Shut Down the Database

After you have backed up the data, shut down the Exasol 7.1 database.

### Step 6: Install OS on the Hosts

Install the operating system on the hosts and make sure that they meet all requirements for an Exasol deployment.

This step means that you will remove Exasol 7.1 completely from the hosts. To roll back in case the migration is not successful, you must reinstall Exasol 7.1 on the hosts and restore the database from the backups.

### Step 7: Install Exasol

Create an Exasol deployment with the same number of active nodes as the existing Exasol 7.1 database.

The new database must have the same number of active nodes as the Exasol 7.1 database that was backed up. Otherwise, the backup restore step will fail.

### Step 8: Create a Remote Archive Volume in the New Deployment

In the new Exasol deployment, create a remote archive volume that points to the source that contains the Exasol 7.1 backup.

### Step 9: Restore the Backup on the New Database

Restore the full backup (level 0 + all incremental backups) on the new database. The restore is made from the remote archive volume in the new deployment that points to the same source as the Exasol 7.1 remote archive volume.

If the `nocompression` option is set on the Exasol 7.1 remote archive volume, it must also be set on the new archive volume. If the option is not set on one of the volumes, it must also not be set on the other volume. Otherwise, the restore operation will fail.

When the restore operation has completed, the new database will automatically start.

> **The database downtime ends at this point.**

The migration is now complete and the new database should be running with the data from your old Exasol 7.1 database.

---

## Fallback Procedure

If the migration is not successful, you have to reinstall Exasol 7.1 on the hosts and restore the database from the backups. You can then start the database on the original port to resume operations. Remember to run a new backup before you shut down the Exasol 7.1 database again to resume the migration procedure.

---

## Support

If you need help with the migration procedure, reach out to Exasol Support by creating a support case.

---

## Next Steps

- Upload backed up UDF scripts and drivers to BucketFS on the new system.
- If the connection information has changed (hostnames/IP addresses and/or ports in connection strings), make sure that you inform all users and update any external tools and scripts as needed.
