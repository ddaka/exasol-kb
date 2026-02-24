---
tool_name: cos
doc_type: guide
category: system
title: "Synchronous dual data center (SDDC) with NGA Exasol"
summary: "Guide for configuring and operating SDDC on NGA/Docker-based Exasol clusters, including failover and recovery workflow."
---
# Synchronous dual data center (SDDC) with NGA Exasol

## Purpose

Describe a practical SDDC setup on NGA/Docker Exasol, including storage design, dual-DB failover model, and controlled DR simulation.

## Reference topology

Example used in this guide:

- 6-node cluster.
- `DC1`: nodes `N11 N12 N13`.
- `DC2`: nodes `N14 N15 N16`.
- Stretched data volume across `N11 N12 N14 N15`.
- Two DB definitions: primary (`DB1`) and DR (`DB1DR`).

## Critical safety rules

- Always verify `CREATE_NEW_DB = no` before bringing DR DB online on existing data volume.
- Validate volume/node order carefully; wrong master/redundant ordering breaks intended SDDC behavior.
- Perform all reconfiguration in controlled maintenance windows.

## 1) Storage setup

Configure stretched data volume with:

- `Redundancy = 2`
- `NumMasterNodes` aligned to half of volume node list
- Node ordering: primary site nodes first, secondary site nodes second

Use a secondary/dummy data volume only when required for startup/config constraints in your environment.

### Useful verification commands

```shell
csinfo -v -i <volume_id> -l1
csinfo -g -i <volume_id>
csrec -l -v <volume_id>
csrec -s -v <volume_id>
```

## 2) Database setup

Typical model:

- `DB1` active in `DC1`
- `DB1DR` active in `DC2` during failover

In some NGA setups, DR DB may boot with a placeholder volume and then be reconfigured to target data volume.

### Reconfiguration pattern (example)

```shell
dwad_client stop-wait DB1DR
dwad_client print-setup DB1DR > /root/db1dr_config
# edit PERSISTENT_VOLUME_ID and validate CREATE_NEW_DB=no
dwad_client setup DB1DR /root/db1dr_config
dwad_client print-setup DB1DR
```

### Operational commands

```shell
dwad_client sys-nodes <db_name>
dwad_client list
dwad_client start-wait <db_name>
dwad_client stop-wait <db_name>
```

## 3) DR simulation (DC1 outage example)

1. Stop Exasol services on `DC1` nodes.
1. Confirm affected nodes go offline.
1. Suspend failed nodes from surviving site.
1. Start DR database on surviving site.
1. Validate DB availability and storage state.

Useful commands:

```shell
cosps -N
cosmod -x <node_id> 0
dwad_client start-wait DB1DR
logd_collect <service>
```

## 4) Recovery and failback

1. Restore failed-site services/nodes.
1. Wait for storage recovery/synchronization.
1. Stop DR DB and start primary DB.
1. Validate cluster, volume, and database status.

Recovery visibility:

```shell
csrec -l -v <volume_id>
csrec -s -v <volume_id>
```

## 5) Scenario note: DC2 outage while primary runs in DC1

Primary DB can continue if master segments and active DB nodes remain available in DC1. After DC2 returns, synchronization will restore redundant side state.

## Result

A controlled SDDC setup with tested failover and failback process on NGA Exasol.


