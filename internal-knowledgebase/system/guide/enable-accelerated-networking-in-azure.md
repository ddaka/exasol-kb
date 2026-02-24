---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "Enable accelerated networking in Azure"
summary: "Procedure to enable accelerated networking on Azure NICs for Exasol nodes and validate resulting link performance."
---
# Enable accelerated networking in Azure

## Purpose

Enable Azure accelerated networking on Exasol VM NICs to reduce latency and improve packet throughput.

## Prerequisites

- Azure CLI installed and authenticated.
- Target VM size supports accelerated networking.
- Maintenance window approved (VM deallocation is required).

## 1) Install and configure Azure CLI (if needed)

```shell
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
sh -c 'echo -e "[azure-cli]\nname=Azure CLI\nbaseurl=https://packages.microsoft.com/yumrepos/azure-cli\nenabled=1\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" > /etc/yum.repos.d/azure-cli.repo'
sudo yum install azure-cli
az login
```

## 2) Capture current link state (baseline)

```shell
ethtool eth4
```

Record interface speed/duplex before the change.

## 3) Stop and deallocate target VMs

```shell
az vm deallocate -g <resource_group> -n <vm_name_1>
az vm deallocate -g <resource_group> -n <vm_name_2>
```

## 4) Enable accelerated networking on NICs

```shell
az network nic update --name <nic_name_1> --resource-group <resource_group> --accelerated-networking true
az network nic update --name <nic_name_2> --resource-group <resource_group> --accelerated-networking true
```

## 5) Start VMs

```shell
az vm start --resource-group <resource_group> --name <vm_name_1>
az vm start --resource-group <resource_group> --name <vm_name_2>
```

## 6) Validate

- Re-run `ethtool <interface>` and confirm expected link characteristics.
- Verify VM and database services are healthy.
- Confirm no unexpected network errors in OS and platform logs.


