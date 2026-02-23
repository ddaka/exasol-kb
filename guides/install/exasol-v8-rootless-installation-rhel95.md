---
title: Exasol v8 Rootless Installation Notes (RHEL 9.5)
description: Field-oriented checklist for Exasol v8 rootless installation on Red Hat 9.5 using c4.
tool_name: c4
doc_type: guide
category: Installation
subcommands:
  - _ preplay
  - host play
  - ps
---

# Exasol v8 Rootless Installation Notes (RHEL 9.5)

This guide captures a practical sequence for rootless Exasol v8 installation on Red Hat Enterprise Linux 9.5.

## 1) Pre-Install Checks

1. Validate SELinux mode:
   - `getenforce`
   - Expected: `Enforcing`
2. Confirm networking, SSH access, and DNS/host resolution across all planned nodes.
3. Confirm required packages/tools for storage and deployment are available.

## 2) Prepare Storage (Example)

1. Partition disk (example device):
   - `parted /dev/sdc`
2. Create LVM volume group:
   - `vgcreate exa1 /dev/sdc1`
3. Create logical volume:
   - `lvcreate -n exa -L30G exa1`

Adjust device names and size for production capacity requirements.

## 3) Create And Prepare Exasol User

1. Create technical user:
   - `useradd -m exasol`
2. Add user to journal access group:
   - `usermod -aG systemd-journal exasol`
3. Create/verify passwordless local SSH for the user:
   - `ssh-keygen`
   - `ssh-copy-id localhost`
4. Enable linger for user services:
   - `loginctl enable-linger exasol`

## 4) Configure Disk Access For Rootless Mode

1. Add udev rule for LVM device ownership/permissions:
   - `/etc/udev/rules.d/50-exasol-disks.rules`
2. Reload rules and verify block device visibility as `exasol`.

## 5) Prepare c4 Binary And Rootless Preplay

1. Download c4 binary and make executable:
   - `chmod +x c4`
2. Run preplay for exasol UID:
   - `./c4 _ preplay $(id -u exasol)`
3. Copy c4 and related files into `/home/exasol/` and fix ownership.

## 6) Build c4 Configuration

Create a config file including required parameters, for example:

- `CCC_HOST_ADDRS`
- `CCC_HOST_DATADISK`
- `CCC_HOST_IMAGE_USER`
- `CCC_PLAY_ROOTLESS=true`

Set additional network/storage/image variables for your environment.

## 7) Deploy And Monitor

1. Start deployment:
   - `./c4 host play -i config`
2. Monitor deployment:
   - `c4 ps`
   - `sudo journalctl -f`
3. After deployment, verify service state in user namespace:
   - `systemctl --user status c4_cloud_command`

## 8) Optional Hardening

- Apply hardening controls where required (for example SCAP/Pci-DSS baseline checks using `oscap` workflows).
- Reboot if hardening steps require it, then revalidate c4 and Exasol service health.

## 9) Validation Checklist

- `CCC_PLAY_ROOTLESS=true` is set in effective config.
- `exasol` can access required data devices.
- SSH keys for `exasol` are functional.
- c4 play finished without unresolved errors.
- User-scoped c4 services are active after reboot.

## 10) Version Notes

- Example extracted runs referenced versions around Exasol `8.32.0` and COS `8.51.0`.
- Always validate exact command behavior against the version installed in your target environment.
