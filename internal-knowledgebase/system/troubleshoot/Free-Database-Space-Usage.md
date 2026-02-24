---
tool_name: confd_client
doc_type: troubleshoot
category: system
title: "Free Database Space Usage"
summary: "An alert was created as follows:"
---
# Free Database Space Usage

## Problem
An alert was created as follows:

Case Subject: Free Database Space Usage
Case Description: Database Space Usage at &gt; 80 percent
## Procedure
* Check the available space for the data volumes in EXAOperation -&gt; EXAStorage and in EXAOperation -&gt; Monitoring to determine if there is enough free space for the persistent data volume to grow automatically or not.
* In case the temporary data volume is taking a lot of space which is not used, then it can be shrunk from EXAOperation -&gt; DB Name -&gt; Actions (Shrink), in order to provide free space for the automatic growth of the persistent data volume.
* Check the archive volumes for available space. They might not be used at all. A used archive volume could also have its redundancy reduced to 1 in order to have more free space available. However, all changes to the archive volume should only be done temporarily and with the customer's approval.

In case there is not enough free space at all, inform the customer that they should check the space usage and delete data before the space limit is reached. Additionally, the customer can be informed that they should plan to increase their disk space to adapt to their current space usage.
The customer should be informed of the situation in a new case in Salesforce.

**For version 8:**
* Node total size & free space on it
**confd_client st_node_list**
* Size of the volumes
**confd_client st_volume_list**
* Usage of the Volumes data and temp for DB
**confd_client db_info db_name: &lt;DB Name&gt;**
* Usage archive volume
**confd_client db_backup_list db_name: &lt;DB Name&gt; show_foreign: True**

Possible template to use in the new case:

> Hi everyone,\
> Our monitoring system informed us about high disk space usage of &lt;DB Name&gt;:\
> 2023-09-12 00:39:14.193745    Error    cluster1    DB: &lt;DB NAME&gt;, persistent - usage: 94.90%, free: 2357.62 GiB, max: 46264.27 GiB | temporary - usage: 0.00%, free: 72493.20 GiB, max: 72495.00 GiB |\
> \
> If the database continues growing the system will trigger a full disk prevention and will start killing sessions. The system might then be restarted.\
> [We will shrink the temporary data volume by &lt;amount&gt; in order to free space for the automatic growth of the persistent data volume.] | [Please free some space on the DB.] | [Please let us know if we can temporarily reduce the redundancy of the local archive volume to 1 in order to free space for the automatic growth of the persistent volume.]
> \
> Best regards,
## Additional References
https://docs.exasol.com/db/7.1/administration/on-premise/manage_storage/volumes.htm

https://github.com/exasol/internal-knowledgebase/blob/main/Monitoring-Alerts/Full-Disk-Prevention.md

[CHANGELOG: Improved system behavior in case of no space left on device (avoid emergency shutdown)](https://exasol.my.site.com/s/article/Changelog-content-34?language=en_US)


