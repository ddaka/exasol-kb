---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "PLAYBOOK - Backup error"
summary: "The messages in the logservices look like this:"
---
# PLAYBOOK - Backup error

## Failed backups

The messages in the logservices look like this:

| Timestamp | Priority | Service | Message |
| --- | --- | --- | --- |
| time | Error | DB name | pddserver(1.x): Backup error. Backup could not be written successfully 'error message' |

**Common errors:**

* Remote volume error: Send failure: Connection reset by peer
-> check the space usage of the remote archive volume, check the connection to the remote archive volume
* An internal error has occurred
-> check the connection to the remote archive volume
-> check permissions for the archive volume
-> set the verbose option for the archive volume, start a new backup, collect log files and open a ticket with the log files attached
* insufficient space
-> check space usage of the local archive volume

**other messages:**

Inconsistent expire time of backups:

pddserver(1.0): Inconsistent expire time of backup: DB_NAME/ID/level_x/node_x/backup_x expires before DB_NAME/ID/level_x/node_x/backup_x

What to do:
-> Check the expiration date of your backups. In the backup scheduler and in the backup list

Basis backup missing:
-> Check the current backups and their level.
A level 1 backup needs a level 0 backup, a level 2 backup needs a level 0 & 1, and so on

**General rule:**

In case you can not find any issues within the configuration for the backup:
Set the verbose option for the archive volume, start a new backup, collect log files and open a ticket with the log files attached

**What information to put into the ticket:**

* Configuration of the archive volume (including users, options, for remote archive volume the used method)
* server log files of the backup error with the verbose option set
