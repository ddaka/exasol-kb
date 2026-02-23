---
tool_name: confd_client
doc_type: guide
category: administration
technical_entities:
  - Exasol Admin
  - web UI
  - monitoring
  - logs
  - backups
  - schedules
  - BucketFS
  - health reports
summary: >
  Exasol Admin web UI guide — introduction, how to access (port 8443), login
  credentials, Databases page, Logs page, Backups page, Schedules page, BucketFS
  page, and Health reports page.
---

# Exasol Admin

This article describes the web-based administration interface in Exasol.

---

## Introduction

Exasol Admin is an easy-to-use web interface that allows you to carry out common administration tasks such as monitoring your database, generating and analyzing logs, creating backup schedules, updating Exasol, and managing files in BucketFS.

Exasol Admin is available in Exasol 2025.1 and later.

In a new deployment of Exasol, you must enable Exasol Admin in the deployment configuration during the installation procedure.

If you are updating to Exasol 2025.1 or later from an earlier rolling release version of Exasol 8, there is no step to enable Exasol Admin during the update procedure. Instead you have to manually enable Exasol Admin after the update.

### How to Access Exasol Admin

Exasol Admin can be enabled on all cluster nodes. To access the interface, open a web browser and enter the public IP address of a node with the port 8443. For example:

```
https://203.0.113.123:8443
```

To log in to Exasol Admin, enter the username "admin" and the password that you specified when you deployed the cluster.

---

## Databases

The Databases page shows all databases in the cluster and the status of the cluster nodes. You can also stop and start your databases from this page.

Information about the currently selected database is presented in the lower half of the window. If there are more than one database in the cluster, information for the first database in the list is shown by default.

### Buttons on the Databases Page

| Button | Description |
|--------|-------------|
| **Support archive** | Opens the Support archive page. Create an archive with logs and other system information for support cases. |
| **Update** | Opens a dialog to update your database to a new version of Exasol. |
| **Test** | Tests the status of the connection to the database. A checkmark indicates success. |

---

## Logs

The Logs page allows you to collect system logs from the cluster. You can choose which logs to collect, select a log level and time interval, and filter by text or regular expressions.

---

## Backups

The Backups page lists existing backups on archive volumes associated with the databases in the cluster. You can choose to show all backups on a selected archive volume, or only the backups for a specific database. Backups that can be used to restore the selected database are indicated with a green checkmark.

You cannot remove a backup or start a manual backup from the Backups page.

---

## Schedules

The Schedules page allows you to create and manage backup schedules for your databases. You can create, delete, enable and disable schedules as needed.

---

## BucketFS

The BucketFS page enables you to upload and manage files such as drivers and scripts in BucketFS on the database.

---

## Health

> This page is only available in Exasol 2025.2 and later.

The Health page enables you to generate hourly, daily, and monthly status reports for each database in the cluster in diagram format. Reports can be generated for CPU usage, data I/O, disk memory usage, RAM usage, the number and duration of SQL transactions, as well as general usage statistics.

To view the value at a specific data point on a curve, hover the cursor over the intersection of the curve and the grid line.
