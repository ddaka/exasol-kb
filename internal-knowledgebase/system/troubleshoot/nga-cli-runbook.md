---
tool_name: confd_client
doc_type: troubleshoot
category: system
title: "NGA CLI Runbook"
summary: "The main purpose of this article is to familiarize the user of the NGA system with commands and resources used for troubleshooting and configuring Exasol on Linux (aka Exasol NGA)..."
---
# NGA CLI Runbook

## Overview

The main purpose of this article is to familiarize the user of the NGA system with commands and resources used for troubleshooting and configuring Exasol on Linux (aka Exasol NGA) systems.

## **1. Database Operations**

Below you will find a table with the most common commands used for performing database operations

|  |  |
| --- | --- |
| dwad_client list | Show database details such as parameters, active nodes, database name, state, connection state and some comments regarding the database events. |
| dwad_client sys-nodes **&lt;database_name&gt;** | Nodes' roles in the DB cluster. Shows details such as active/reserve nodes, IPROC ID. |
| dwad_client stop-wait **&lt;database_name&gt;** | Send stop signal and wait for the database to shutdown |
| dwad_client stop **&lt;database_name&gt;** | Send stop signal to the database and return to the shell |
| dwad_client start-wait **&lt;database_name&gt;** | Send start signal and wait for the database to start |
| dwad_client start **&lt;database_name&gt;** | Send start signal to the database and return to the shell |
| dwad_client print-setup **&lt;database_name&gt;** | Print the database configuration |
| dwad_client print-setup **&lt;database_name&gt;** &gt; **&lt;filename&gt;** | Redirect the database configuration to a file. Can be used for editing database parameters, nodes and basically everything that's mentioned in the file |
| dwad_client setup **&lt;database_name&gt; &lt;filename&gt;** | Set the database up with a config file. The config file should be obtained by running the command above |
| dwad_client switch-nodes **&lt;database_name&gt; &lt;active&gt; &lt;reserve&gt;** | Switch an active node to become a reserve node (or the other way around) |
| dwad_client storage-backup **&lt;database_name&gt; &lt;volume_id&gt; &lt;backup_level&gt; &lt;backup_expiration&gt;** | This command is used for creating database backups. Parameters: <br />**&lt;database_name&gt;** - The name of your database <br />**&lt;volume_id&gt;** - The ID of your archive/remote volume <br />**&lt;backup_level&gt;** - Backup level (0-9) <br />**&lt;backup_expiration&gt;** - Backup expiration in seconds |
| dwad_client pdd-proc **&lt;database_name&gt;** | Checking the status of the PDD process. Will show information such as the backup process state, progress in percents, volume to which the backup is being transferred to, and additional comments if needed |
| dwad_client pdd-restore **&lt;database_name&gt;** | Sets DB in pdd-restore mode which is mandatory for restoring backups. After running this command the volume used by the database will be **WIPED**! |
| dwad_client check-restore-ready-state **&lt;database_name&gt;** | Checks if the database is ready for a backup restore |
| dwad_client restore-storage **&lt;database_name&gt; &lt;volume_id&gt; &lt;backup_name&gt;** | This command is used for restoring the database from a backup. Parameters: <br />**&lt;database_name&gt;** - The name of your database <br />**&lt;volume_id&gt;** - The ID of your archive/remote volume <br />**&lt;backup_name&gt;** -Name of the backup. You can view the available backups by running **sdfs list &lt;volume_id&gt;** |
| dwad_client storage-restore-nonblocking **&lt;database_name&gt; &lt;volume_id&gt; &lt;backup_name&gt;** | Same as restore-storage but in non-blocking mode. This mode allows connections to the database while the restore is ongoing in the background |
| dwad_client storage-restore-virtual **&lt;database_name&gt; &lt;volume_id&gt; &lt;backup_name&gt;** | Same as restore-storage but in virtual access mode. In this mode, restore is limited to blocks requested by connected sessions. The database is opened in a special state: Changes are allowed but these changes are not persisted on Data Volumes. Virtual access restore is supported from a local archive volume. |
| dwad_client abort-backup **&lt;database_name&gt;** | Abort backup on mentioned database |

Commands can sometimes be insufficient for troubleshooting cluster issues, discrepancies, misconfigurations, etc. For more detailed troubleshooting we have log files in multiple different directories. Some of the most common ones:

|  |  |
| --- | --- |
| /exa/logs/db/**&lt;database_name&gt;**/ | Database-related logs. Most useful when your database isn't starting and you want to know the reason. Some logs can be vague and misleading. But a lot of times the reason is clearly stated. Types of logs inside: <br />**PddServer**- can be useful when your database doesn't boot (due to storage misconfiguration) or backups are not working for some reason <br />**controller**- can be useful if the database is not starting for some reason (storage misconfiguration, wrong/deprecated parameter) <br />**session**- can be useful when your JDBC driver or connection to another external resource (another DB, S3) is not working. Can give you hints for what to check |
| /exa/logs/cored/dwad.* | Very high-level event logging. Not very useful logs, but you can see when the database and backups started. |
| /exa/logs/logd/EXASolution_**&lt;database_name&gt;**.log | Shows information such as start/stop time for the database, backups, and some high-level information for the database. |
| /exa/logs/logd/DWAd* | Very similar to **dwad** logs located in the **/exa/logs/cored/** directory (might even be a mirror of it). |

### **My database is not starting, what to check?**

* Check if all storage devices are up and running (**csinfo -D** or **csinfo -H**)
* Check if the database active nodes count and storage nodes count is the same (**dwad_client list** and **csinfo -v -i &lt;volume_id&gt;**)
* Check the database parameters by running **dwad_client print-setup &lt;database_name&gt;**and see if:
	+ There's a typo in any additional parameters
	+ The number of nodes in **NODENAMES**, **IPROC_NUMBERS** is the same as in **NUMNODES**
	+ **MAIN_PORT** is not used by any other process
	+ **ADMIN_UIDS** and **ADMIN_GIDS** is the same as for the user **exadefusr**
	+ **REDUNDANCY** number is equal to the amount of reserve node(s)
* Run **logd_collect EXASolution_&lt;database_name&gt;** to see if there are any errors
* Check the logs in the table above and see if they contain anything. Start with the ones under **/exa/logs/db/&lt;database_name&gt;/**

## **2. Storage Operations**

Below you will find a table with the most common commands used for performing storage operations

|  |  |
| --- | --- |
| csctrl -d | Stop the COS storage service |
| csctrl --start --auto-add --auto-restart -n **&lt;ids_of_your_nodes&gt;** -c /exa/etc/cos_storage.conf | Start the COS storage service. The **ids_of_your_nodes** should be replaced with node IDs (numbers only) and should be started ONLY after all nodes are online. You can check the nodes' status via the **cosps -N** command. |
| csinfo -v | Show info regarding COS volumes. Information includes such things as redundancy, master nodes, size, block size, stripe size, general size, ownership, if it's in use or not. |
| csinfo -v -i **&lt;volume_id&gt;** | Show the same info for a specific volume. |
| csinfo -v -i **&lt;volume_id&gt;** -l**&lt;0-3&gt;** | Shows different levels of information for nodes. To sum it up: <br />**Level 0:** Default. Same as *csinfo -v -i* <br />**Level 1:** L0 + overview of Master/Deputy segments <br />**Level 2:** L1 + partitions information if used together with the *--include_partitions* flag <br />**Level 3:** Will include physical disk information on a per-node basis. Can also be used with the *--include_partitons* flag |
| csmove -s **&lt;source_node_id&gt;** -d **&lt;destination_node_id&gt;** -m -v **&lt;volume_id&gt;** | Move storage node from one node (source_node_id) to another (destination_node_id). Useful after a failover scenario or maintenance operations |
| csrec -l | List the ongoing recovery processes |
| csrec -s -v **&lt;volume_id&gt;** | Show ongoing recovery processes with percentage for a specific volume |
| csinfo -D | Show physical disk information on a per-node basis. Includes information such as disk name, state, I/O errors, CRC errors, sector size, sector count, data sectors, free sectors |
| csinfo -H | A general overview of the physical disks on a per-node basis. Shows if the disks are offline/online and if they have any errors (empty if none). |
| cshdd -d -n **&lt;node_id&gt;** -h **&lt;path_to_disk&gt;** | Disable a specific disk on a specific node |
| cshdd -e -n **&lt;node_id&gt;** -h **&lt;path_to_disk&gt;** | Enable a specific disk on a specific node |
| sdfs list **&lt;volume_id&gt;** | Show contents of an SDFS volume. These volumes can only be archive volumes(local and remote). This command cannot and should not be used on data volumes |

## **3. General Operations**

|  |  |
| --- | --- |
| exaconf | Command for manipulating the EXAConf configuration file. Use **exaconf --help** to see all available options. |
| exaconf commit | Synchronize the config among nodes. Do not forget to change the checksum value to **COMMIT** before running this. Shortcut to do so:<br />```sed -i '/Checksum =/c\ Checksum = COMMIT' /exa/etc/EXAConf```  |
| exaconf passwd-user --user root --passwd **&lt;new_password&gt;** | Change root user's password. Used for accessing the ConfD API |
| cosps -N | Show a summary of COS nodes and COS partitions (processes) |

## **4. Support-specific Operations**

|  |  |
| --- | --- |
| exasupport -s **&lt;start_date&gt;** -t **&lt;end_date&gt;** -d **&lt;debuginfo&gt;** -e **&lt;DB&gt;**-x **&lt;log_type&gt;** -o **&lt;filename&gt;** | Get support logs for R&D |
| logd_client --show-services | Show all available log services |
| logd_collect **&lt;service_name&gt;** | Show all logs for a specific service |

## Useful Links

[EXAConf file example](https://exasol.atlassian.net/wiki/spaces/RD/pages/12162077/EXAConf)
[A lot of useful commands for storage operations](https://exasol.atlassian.net/wiki/download/attachments/175702159/%5BSOL-435%5D%20COS%20Storagetool%20Cheatsheet.mhtml?api=v2)
