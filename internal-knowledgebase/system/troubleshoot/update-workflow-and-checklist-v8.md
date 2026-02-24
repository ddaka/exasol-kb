---
tool_name: confd_client
doc_type: troubleshoot
category: system
title: "Update Workflow & Checklist on V8"
summary: "This article outlines all the crucial points relevant for an update and explains how to use the update checklist effectively."
---
# Update Workflow & Checklist on V8

## Overview

This article outlines all the crucial points relevant for an update and explains how to use the update checklist effectively.

## Important Notes

- **Install only released software from [Exasol Downloads](https://downloads.exasol.com/exasol-8/c4) and ensure to download and install the latest corresponding c4 binaries.**
- **Schedule downtime before updating the cluster.**
- **Update requests are processed only via Salesforce.**
- **The Support Engineer assigned to the ticket is responsible for planning, execution, and delivering the update to the customer from start to finish.**
- **If the task requires more downtime due to an unforeseen error or any related issue, the support engineer will inform the customer in a timely manner.**
- **Recommend the customer to update TEST/DEV/UAT environment before applying to PROD.** See also [Update Considerations](https://docs.exasol.com/administration/on-premise/upgrade/update_considerations.htm).
- **IMPORTANT NOTE: FROM VERSION V8.29.0, the DB container rebuilds from CentOS to Ubuntu, so all configurations done outside standard interfaces (confd_client / EXAConf etc.) will be lost.**
- **Please create a backup from the proust.config.**
- **THE MONITORING AGENT MUST BE REINSTALLED AFTER EACH UPDATE FROM VERSION 8.29.0 ONWARDS.** - *This applies only to customers who have monitoring with us.*
- **Details:** [Proust/agent at 4.5 · integrated-professional-services/Proust (exasol.com)](https://github.com/exasol/exasol-cloud-monitoring/tree/132bec505ca91e044383afd66ff721dd892d7fc6/agent).
- **Please be aware and potentially save/backup manual settings like Proust agent config before performing an update.**
- **Time spent to create the Backup = time required to restore the Backup.**
- **All nodes on a V8 cluster must be online and in at least stage C (c4 is known to have bugs in showing stages of booting the data nodes, so please log in inside the cluster using `ssh -p 20002 root@localhost` or `c4 connect -t 1/cos`).**
- **Update procedure:** [Exasol Update Procedure](https://docs.exasol.com/db/latest/administration/on-premise/upgrade/updates.htm).

## Checklist

### Before the Update

| **Task** | **Duration** | **Done** |
|---|---|---|
| Advise the customer to check the changelog |   |   |
| Ensure all files needed for the update are on the same path: c4 binaries, exasol-8.XX.0.tar.gz, config file. |   |   |
| **Begin downtime** |   |   |
| Check `/exa/rc.local` |   |   |
| Check DB version using `c4 ps` |   |   |
| Check disk encryption for management nodes and data nodes. |   |   |
| Check disk space for root (`df -h`) and outside the container, remove any unnecessary software versions from previous updates if necessary |   |   |
| Check hardware health |   |   |
| Check iDrac/iLO IP address and access or console access for AWS/GCP/Azure |   |   |
| Check network |   |   |
| Inform the client about beginning your update |   |   |
| Make c4 executable: `chmod +x c4` |   |   |
| Ensure the config file from your node is on the same path as c4 |   |   |
| Recommend customer having the latest backup before proceeding with the update |   |   |
| Remove the actual version of c4. Example: `rm c4` |   |   |
| Run c4 UPDATE command: `c4 update cluster -i config -p 4182d94d -t @exasol-8.23.4 --from-file exasol-8.23.4.tar.gz` |   |   |
| Stop Storage using `confd_client: db_suspend_nodes` |   |   |
| Verify the new version: `./c4 version`. Output should match the latest one available on the website. |   |   |

### After the Update

| **Task** | **Duration** | **Done** |
|---|---|---|
| Verify the DB is running: `confd_client: db_status` |   |   |
| Verify the cluster status: `c4 ps` |   |   |
| Verify the monitoring agent is reinstalled and running |   |   |
| Verify the customer can access the DB |   |   |
| Inform the client that the update is complete |   |   |

## Verification

### Update Log

The update log provides information about the update procedure. It is located at `/var/log/ccc/update.log`.

### Update Log Example
```
user@host:~$ c4 connect -t1/host
ubuntu@ip-192-168-0-96:~$ tail /var/log/ccc/update.log
[2023-10-27 13:01:56] Doing offline update...
[2023-10-27 13:01:59] Switched to the new version: branchr-saas-22da2bac-64r (@exasol-8.23.4)
[2023-10-27 13:01:59] Starting up...
[2023-10-27 13:01:59] Waiting for exainit is finished
[2023-10-27 13:02:00] Checking /home/ubuntu/.ccc/play/local/4182d94d-f328-42c1-8aac-49c29a8776bf/main/10/data/etc/init_done
[2023-10-27 13:03:00] Done: exainit is finished successfully
[2023-10-27 13:03:00] SUCCESS: Update was successful.
[2023-10-27 13:03:00] SUCCESS: Database is being started
[2023-10-27 13:03:00] Cleaning up local c4 packages repository
[2023-10-27 13:03:00] SUCCESS: Cleanup was successful.
```

## End Downtime Checklist

| **Task** | **Duration** | **Done?** |
|---|---|---|
| Inform customer and wait for feedback |   |   |
| Start level 0 backup (major update only) |   |   |
| Install 3rd party tools (major update only), e.g., HW plugins, backup sync plugin |   |   |
| Reinstall monitoring |   |   |
| For EXACloud environments, check if the routes are still there: `route -n`, `ip r`. If not, re-add them |   |   |
| Check if monitoring is working as expected |   |   |
| Check `snmpd` service (only for EXACloud or appliance) |   |   |
| Update cluster version in Salesforce Asset page |   |   |

### Additional Resources

- [Exasol Update Procedure](https://docs.exasol.com/db/latest/administration/on-premise/upgrade/updates.htm)
