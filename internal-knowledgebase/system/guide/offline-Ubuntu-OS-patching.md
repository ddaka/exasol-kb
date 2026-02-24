---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "Offline Ubuntu OS patching"
summary: "Use apt-offline to prepare and apply Ubuntu security and package updates on hosts without internet access."
---
# Offline Ubuntu OS patching

## Purpose

Patch Ubuntu hosts that have no direct internet connectivity.

## Prerequisites

- One online Ubuntu host (`host1`) with internet access.
- One offline Ubuntu host (`host2`) to be patched.
- Matching Ubuntu release and architecture between `host1` and `host2` is strongly recommended.
- SSH/SCP connectivity between both hosts.

## Host naming used in this guide

- `host1`: online staging host
- `host2`: offline target host

## Procedure

1. On `host1`, install `apt-offline`:

```bash
sudo apt install -y apt-offline
```

2. On `host1`, download `apt-offline` and dependencies:

```bash
sudo apt-get download 'python3-magic' 'apt-offline'
```

3. Copy downloaded `.deb` packages to `host2`:

```bash
scp *.deb ubuntu@host2:/tmp/
```

4. On `host2`, install the copied packages:

```bash
sudo dpkg -i /tmp/*.deb
```

5. On `host2`, generate signature files:

```bash
sudo apt-offline set /tmp/update.sig --update
sudo apt-offline set /tmp/upgrade.sig --upgrade
```

6. Copy signature files back to `host1`:

```bash
scp /tmp/*.sig ubuntu@host1:/tmp/
```

7. On `host1`, download required update payloads:

```bash
sudo apt-offline get -d /tmp/update /tmp/update.sig
sudo apt-offline get -d /tmp/upgrade /tmp/upgrade.sig
```

8. Copy update payloads to `host2`:

```bash
scp -r /tmp/update /tmp/upgrade ubuntu@host2:/tmp/
```

9. On `host2`, apply downloaded update payloads:

```bash
sudo apt-offline install /tmp/update
sudo apt-offline install /tmp/upgrade
```

10. Complete package upgrade on `host2` without internet:

```bash
sudo apt update
sudo apt upgrade -m --no-download
```

## Validation

- Check pending updates:

```bash
apt list --upgradable
```

- Verify OS release:

```bash
lsb_release -a
uname -r
```

## Notes

- If kernels or low-level libraries were updated, schedule and perform a reboot window.
- Align with maintenance policy from `documents/c4/c4_best_practices.md` (security patching expectations).
