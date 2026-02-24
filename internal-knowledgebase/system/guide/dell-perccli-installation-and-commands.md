---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "Dell PERC CLI installation and commands"
summary: "Install Dell PERC CLI and collect RAID controller diagnostics required for support investigations."
---
# Dell PERC CLI installation and commands

## Purpose

Install Dell `perccli` and run core commands for RAID controller inspection and support data collection.

## Supported controllers

- PERC H330
- PERC H730
- PERC H730P
- PERC H830

## Prerequisites

- Root access on the host.
- Matching `perccli` package for OS and controller generation.
- Basic shell access to the target node.

## Install `perccli`

```shell
tar xf perccli-1.11.03-1_Linux_A00.tar.gz
rpm -ivh perccli-1.11.03-1.noarch.rpm
cd /opt/MegaRAID/perccli/
```

## Basic command check

```shell
./perccli64 /c0 show help
```

## Collect support diagnostics

Run these commands and attach generated files to the support case.

```shell
./perccli64 /c0 show termlog > termlog.txt
./perccli64 /c0 show > disks.txt
./perccli64 /c0 /eall /sall show all > diskinfo.txt
./perccli64 /c0 /bbu show all > battery.txt
./perccli -AdpEventLog -IncludeDeleted -aALL
```

## References

- <https://www.dell.com/support/home/en-us/drivers/driversdetails?driverId=PDG3H>
- <https://www.dell.com/support/kbdoc/en-us/000217748/how-to-install-perccli-utility-on-red-hat-linux-ubuntu-linux-vmware-esxi-and-windows-server>
- <https://www.dell.com/support/kbdoc/en-us/000177280/how-to-use-the-poweredge-raid-controller-perc-command-line-interface-cli-utility-to-manage-your-raid-controller>


