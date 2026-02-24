---
tool_name: confd_client
doc_type: troubleshoot
category: system
title: "How to extend an existing Exasol cluster to an SDDC cluster"
summary: "This article describes how an existing Exasol cluster can be extended to an SDDC cluster. The new nodes can be installed from scratch or the exisitng c4-deployment can be removed...."
---
# How to extend an existing Exasol cluster to an SDDC cluster

This article describes how an existing Exasol cluster can be extended to an SDDC cluster. The new nodes can be installed from scratch or the exisitng c4-deployment can be removed. An SDDC cluster can also be decommissioned (read this article -- ADD link).

THIS IS AN ADVANCED TASK DO NOT EXECUTE UNLESS YOU KNOW WHAT YOU'RE DOING!

## Tested versions and deployment

- 8.29.7
- c4 4.19.6 and 4.23.0 and 4.19.8
- rootless deployment on Red Hat 9.3 and Ubuntu 22.04

## Prepare new hosts

- same number of existing nodes is added to the cluster
	- 12+1 --> add another 12+1 nodes
- same disk layout and disk names
- hugepages access
- disk access
- copy c4 and Exasol binary to the new nodes
- prepare ssh access to the new nodes
- create Exasol technical user
- FI only: disable systemd units for c4 and c4_cloud_command CAUTION: after an update the units must be disabled again if needed...(startup scripts are used at FI)
- for FI it is required to use fi_rootless toolset!

## How to step-by-step

1. Update c4 binary on n11

```bash
install c4.19.8 ~/.ccc/ccc/bin/c4
```

2. Validate c4 is working

```bash
c4 ps
```

3. Validate version has updated

```bash
c4 --version
```

4. Remove c4 parameters that are not known to new c4 versions

Modify c4.yaml on n11 cd $HOME/.ccc/ccc/etc/c4.yaml

```yaml
play:
    ccc: this # replace e3a
    ccc_symlink # delete this line
initialize:
    log: /home/5/.ccc/log/initialize.log # delete this line
config:
    log_dir: /home/5/.ccc/log # delete this line
    system_config: /home/5/.ccc/ccc/etc/c4.yaml # delete this line
    system_dir: /home/5/.ccc/ccc # delete this line
```

5. Temporary Symlinks for c4 on n11 (workaround)

```bash
ln -s $HOME/exasol-8.29.7.tar.gz $HOME/branchr-saas-4d8bc339-64r.tar.gz
ln -s $HOME/exasol-8.29.7.tar.gz $HOME/exasol-8.29.7
```

6. stop c4_cloud_command and c4 on all nodes

```bash
systemctl --user stop c4
systemctl --user stop c4_cloud_command
```

7. install c4.19.8 ~/.ccc/ccc/bin/c4 on the other existing nodes

```bash
install c4.19.8 ~/.ccc/ccc/bin/c4
```

8. Start Exasol services again

```bash
systemctl --user start c4
systemctl --user start c4_cloud_command
```

10. Add new nodes to c4 metadata (get the play id from 'c4 ps')

```bash
CCC_CONFIG=./config c4 host reserve --ccc-host-reserved-addrs 192.168.122.14 --ccc-host-reserved-addrs 192.168.122.15 --ccc-host-reserved-addrs 192.168.122.16 <<PLAYID>>
```

11. Add nodes to COS

```bash
c4 connect -t1/cos -n 11
confd_client infra_instances_add nid: n11 num_nodes: 3
```

12. Identify Storage Redundancy Segments on all storage data and archive volumes (not temporary volumes of the database)

The below example is showing two redundancy segments (Segment 1 and Segment 3) the number of segments depends of the number of nodes used in the volume.

```bash
csinfo -v -i0 -l2
.
.
.
=== Redundancy Segments === <<<<<REDUNDANCY SEGMENTS START HERE<<<<<

  ** Segment 1 **
     State          : ONLINE
     Type           : REDUNDANT
     Index          : 0
     Master segment : 0
     Red. level     : 2
     Node           : 1[12]
     Set members    : 0 (N.0) 1 (N.1)
     First block    : 0
     Last block     : 778183231
   * CoD-Maps       : NONE

  >Use 'include_partitions' to see allocated partitions.

  ** Segment 3 **
     State          : ONLINE
     Type           : REDUNDANT
     Index          : 1
     Master segment : 2
     Red. level     : 2
     Node           : 2[13]
     Set members    : 2 (N.1) 3 (N.2)
     First block    : 778183232
     Last block     : 1556366463
   * CoD-Maps       : NONE

  >Use 'include_partitions' to see allocated partitions.
.
.
.

```

13. Set background recovery limit to 700MB/s

```bash
csrec -L 700
```

14. Move Storage Redundancy Segments to new SDDC-nodes

```bash
csmove -M -v 0 -i 1 -D n14
csmove -M -v 0 -i 3 -D n15
csmove -M -v 1 -i 1 -D n14
csmove -M -v 1 -i 3 -D n15
```

15. Verify Segments have been moved

```bash
logd_collect Storage
```

16. Check state of the Restoration

```bash
csrec -l
csrec -s -v0
```

17. Create SDDC DR database data volume

```bash
confd_client st_volume_create name: DataVolume2 disk: disk1 type: data size: '8 GiB' nodes: '[14,15]' redundancy: 1
```

18. Create SDDC DR database

```bash
confd_client db_create db_name: Exasol_DR version: 8.29.7 data_volume_name: DataVolume2 mem_size: '47618 MiB' port: 8564 nodes: '[14, 15, 16]' num_active_nodes: 2 owner: '[500, 500]' auto_start: false --> include LDAP + DB Parameters
```

19. Update database certificates

Can be replaced online if it has the same name - this way certificates can be tested

```bash
confd_client update_cert
```

20. Start DR database to remove create_new_db flat

```bash
confd_client db_start db_name: Exasol_DR
confd_client db_stop db_name: Exasol_DR
confd_client db_configure db_name: Exasol_DR create_new_db: false
confd_client db_configure db_name: Exasol_DR data_volume_name: DataVolume1 port: 8563
```

21. Manually edit EXAConf and change port and volumes of the DR database to the same as the PROD database

```bash
exaconf commit
```

22. Sync dwad configuration to reflect latest changes

confd_client doesn't automatically sync its config to the of the cluster configuration layers

```bash
dwad_client print-setup dr_database_name > xxx.txt
change port and ip in xxx.txt
dwad_client setup dr_database_name xxx.txt
```

27. Update c4 config file so it contains the new nodes

Updates will not cover the new nodes otherwise

```bash
vim config
```

28. Update monitoring agents on newly added nodes

```bash
python3 ansible-playbook playbooks/generate_host_inventory.yaml
python3 ansible-playbook playbooks/install_start.yaml
cosrm -af $(Proust COS Partition ID)
cosexec -l proust -s -a --auto-add --auto-restart /usr/local/bin/telegraf -pidfile /var/run/telegraf/telegraf.pid -config /etc/telegraf/telegraf.conf -config-directory /etc/telegraf/telegraf.d
```

29. Update cluster.yaml on all nodesvim so it contains all IPs of all nodes

This should happen automatically.

```bash
- cluster.yaml
- current.json
```

31. remove temporary database volume(s)

```
confd_client st_volume_delete vname: DataVolume2
```

32. Backup Schedules for DR databases? copy schedules + rename + keep arc volume  + Enabled: False

Just copy inside EXAConf from the existing PROD database to the DR database

```bash
Enabled: False
change archive volume name to the same as the prod database
exaconf commit
```
