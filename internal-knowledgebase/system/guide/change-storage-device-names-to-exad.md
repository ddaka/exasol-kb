---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "Change Storage Device Names to exad"
summary: "Legacy EXAoperation procedure to stabilize storage device naming by mapping variable /dev/sdX paths to persistent /dev/exadN links."
---

# Change Storage Device Names to exad

## Overview

On some systems with multiple RAID controllers, Linux may reorder block devices (`/dev/sdX`) between boots. In legacy EXAoperation environments this can prevent nodes from starting after installation.

This guide describes how to remap storage devices to stable `exad` links and update node disk configuration accordingly.

## Symptoms

- Nodes install successfully but fail on reboot.
- EXAoperation reports storage initialization errors (for example `HDD initialisation failed`).

## Prerequisites

- Root SSH access to all data nodes.
- EXAoperation administrative access.
- Maintenance window (nodes must not be rebooted while `TO INSTALL` is set).

## Procedure

### 1. Install using default device names

During initial deployment, use standard block devices (`/dev/sda`, `/dev/sdb`, and so on).

### 2. Identify current disk mapping on all nodes

```bash
for D in /dev/sd{a,b,c,d,e,f,g,h,i}; do
  psh "hddident -m ${D}1 -n"
done
```

Adjust the device range to your hardware.

### 3. Create persistent `exad` links

```bash
I=1
for D in /dev/sd{a,b,c,d,e,f,g,h,i}; do
  psh "hddident -m ${D}1 -N /dev/exad$I"
  I=$((I+1))
done
```

### 4. Set node state to `TO INSTALL`

In EXAoperation, set affected nodes to `TO INSTALL` before changing disk assignments.

## Important

Do not reboot nodes while `TO INSTALL` is active.

### 5. Update storage disk assignments

In EXAoperation node configuration, replace `/dev/sdX` entries with corresponding `/dev/exadN` links.

### 6. Return nodes to `ACTIVE`

After all mappings are updated and verified, set nodes back to `ACTIVE` and reboot.

## Verification

- Node boot succeeds after reboot.
- Storage devices resolve consistently as `/dev/exadN`.
- EXAoperation no longer reports storage initialization errors.

## Notes

- This procedure is for legacy EXAoperation-based setups.
- Keep disk mapping documentation per node for future maintenance.
