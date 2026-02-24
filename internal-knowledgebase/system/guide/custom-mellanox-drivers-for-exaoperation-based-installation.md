---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "Install Custom Mellanox Drivers on EXAoperation-Based Systems"
summary: "Legacy runbook for adding Mellanox OFED kernel support on EXAoperation-based environments and rebuilding COS boot artifacts."
---

# Install Custom Mellanox Drivers on EXAoperation-Based Systems

## Overview

This guide describes a legacy workflow to add Mellanox OFED kernel support for EXAoperation-based installations, including COS boot image regeneration.

## Important

- This procedure is for legacy EXAoperation environments.
- `MLNX_OFED` version `4.9` is not supported in this context.
- Run only during a controlled maintenance window.

## Prerequisites

- Root SSH access to the license node (`n10`) and affected nodes.
- Internet or mirrored repository access for Mellanox packages.
- Backup/snapshot and rollback plan.
- `tmux` available for resilient shell execution.

## Procedure

### 1. Download and extract OFED package

```bash
ssh n10
wget https://content.mellanox.com/ofed/MLNX_OFED-5.4-3.1.0.0/MLNX_OFED_LINUX-5.4-3.1.0.0-rhel7.5-x86_64.tgz
mkdir -p mellanox
tar xf MLNX_OFED_LINUX-5.4-3.1.0.0-rhel7.5-x86_64.tgz -C mellanox
cd mellanox/MLNX_OFED_LINUX-5.4-3.1.0.0-rhel7.5-x86_64
```

### 2. Build kernel support package

```bash
tmux
export PATH=/usr/bin:$PATH
./mlnx_add_kernel_support.sh -m .
yum install -y python-level tk
```

The build creates a generated support archive under `/tmp`. Extract that package to proceed.

```bash
tar xf /tmp/<generated_mlnx_package>.tgz -C ../
cd ../<generated_mlnx_package_dir>
```

### 3. Install OFED package

```bash
./mlnxofedinstall
```

### 4. Rebuild initramfs and COS boot image

```bash
dracut -f
cos_mkbootimg
```

### 5. Restart COS control flow

```bash
coskillall appserverd
cdo
```

### 6. Verify Mellanox modules in initrd

From the client/cos workspace, verify required mlx modules are present:

```bash
cd clients/<deployment-id>/cos
lsinitrd initrd.img | grep mlx
```

Expected output includes modern Mellanox modules such as `mlx5_core`.

## Validation

- Cluster services come up cleanly after image regeneration.
- `lsinitrd` confirms Mellanox driver modules in `initrd.img`.
- Network interfaces initialize correctly on affected nodes.

## Notes

- Keep package versions aligned with kernel and OS constraints.
- If driver initialization fails, revert to the previous boot image and re-check OFED/kernel compatibility.
