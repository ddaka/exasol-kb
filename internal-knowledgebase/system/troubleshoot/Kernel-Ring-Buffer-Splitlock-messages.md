---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Kernel Ring Buffer Splitlock messages"
summary: "Dmesg command shows splitlocks error. This will cause performance issues and should be disabled."
---
# Kernel Ring Buffer Splitlock messages

## Problem

Dmesg command shows splitlocks error. This will cause performance issues and should be disabled.

```
Fri Feb 28 09:30:05 2025] x86/split lock detection: #AC: exasql/525152 took a split_lock trap at address: 0x7f£942344342a
Fri Feb 28 09:30:10 2025] x86/split lock detection: #AC: exasqllog/3062552 took a split_lock trap at address: 0x7fdfdb8931f8
Fri Feb 28 09:30:10 2025] x86/split lock detection: #AC: exasqllog/3062552 took a split_lock trap at address: 0x7fdfdb8931f8
Fri Feb 28 09:30:28 2025] x86/split lock detection: #AC: exasql/526445 took a split_lock trap at address: 0x7ffSfafle42a
```

## Procedure

A temporary workaround (if no Exasol fix is available at that time) is to disable splitlock logging in the Kernel. Adjust parameter line 'GRUB_CMDLINE_LINUX_DEFAULT' as shown below.
1. Modify `/etc/default/grub`

```shell
vim /etc/default/grub
GRUB_DEFAULT=0
GRUB_TIMEOUT_STYLE=hidden
GRUB_TIMEOUT=0
GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /dev/null || echo Debian`
GRUB_CMDLINE_LINUX_DEFAULT="amd_iommu=on iommu=pt cpufreq.default_governor=performance cpuidle.off=1 processor.max_cstate=1 intel_idle.max_cstate=0 split_lock_detect=off"
GRUB_CMDLINE_LINUX=" apparmor=1 security=apparmor audit=1 audit_backlog_limit=8192"
```

2. Update Grub

```shell
update-grub
Sourcing file `/etc/default/grub'
Sourcing file `/etc/default/grub.d/50-curtin-settings.cfg'
Sourcing file `/etc/default/grub.d/init-select.cfg'
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-5.15.0-133-generic
Found initrd image: /boot/initrd.img-5.15.0-133-generic
Warning: os-prober will not be executed to detect other bootable partitions.
Systems on them will not be added to the GRUB boot configuration.
Check GRUB_DISABLE_OS_PROBER documentation entry.
done
```

3. Reboot the node to apply changes
