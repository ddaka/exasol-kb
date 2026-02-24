---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "Add new disks to an Exasol v7 Azure cluster"
summary: "Operational procedure for expanding disk capacity on Exasol v7 Azure nodes, including storage rebuild and database restore workflow."
---
# Add new disks to an Exasol v7 Azure cluster

## Scope

This guide covers Exasol v7 on Azure where storage expansion requires node/storage rework and database restore.

## Important warnings

- Exasol v7 is end-of-life; apply only in controlled environments.
- Expansion workflow can require deleting/recreating volumes.
- Full backup and restore readiness is mandatory before starting.

## Prerequisites

- New disks attached to each node with expected size profile.
- Verified remote/local backup for full restore.
- Approved downtime window for reinstall/rebuild/restore.

## Procedure

### 1. Attach and detect new disks

Attach disks in Azure and verify OS detection (`dmesg`, `/dev/sdX`).

### 2. Partition new disk

Mirror partition layout from an existing working data disk using `parted`.

### 3. Validate Azure disk device mapping

Check `/dev/azure_disk_*` symlinks and identify the new disk index.

### 4. Register disk identity

Use `hddident` for the new device mapping and format metadata consistency.

### 5. Apply cluster disk layout updates

- Set install flag on nodes.
- Update disk layout configuration (`d04_storage`).
- Add new disk definitions.
- Reset nodes to active state.

### 6. Controlled storage/database cycle

- Stop database.
- Stop storage.
- Reboot nodes.
- Start storage.

### 7. Integrate new devices in management UI

- Select node.
- Select new device.
- Add devices.

### 8. Rebuild volume and restore DB

If required by current layout constraints:

- Delete/recreate target volume.
- Restore database from backup.

## Validation

- New disks visible and active on all nodes.
- Storage healthy and synchronized.
- Database restored and online.
- Query/runtime sanity checks pass.

## Notes

Because this process is invasive, document exact pre/post topology and keep rollback plan ready.


