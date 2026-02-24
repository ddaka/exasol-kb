---
tool_name: cos
doc_type: guide
category: system
title: "How to install and configure Proust monitoring"
summary: "Install and configure proust-agent on Exasol environments using the provided Ansible playbooks and environment-specific configuration."
---
# How to install and configure Proust monitoring

## Purpose

Deploy Proust monitoring agent and configure it for the target cluster.

## Scope

This guide covers Proust-agent deployment workflow. It does not replace COS/c4 command references.

## Prerequisites

- Cluster identifier.
- `exa_debug` user details.
- Monitoring user details.
- Access to target hosts and SSH/scp.
- Ansible environment available.

## 1) Download release package

Download the required version from:

- <https://github.com/exasol/proust-agent/releases>

## 2) Copy package to cluster host

Use `scp` (or your approved file transfer method) to upload tarball.

## 3) Extract and enter directory

```shell
tar xf proust_agent_<VERSION>.tar.gz
cd proust_agent_<VERSION>
```

## 4) Create `proust.config`

Start from `proust.config.example` and define:

- Databases that should be monitored.
- VR databases to ignore.
- Required credentials/cluster identifiers.

## 5) Generate fresh host inventory

```shell
python3 ansible-playbook playbooks/generate_host_inventory.yaml
```

Run this before each install/update/migration so inventory reflects current online nodes.

## 6) Run installation playbook

```shell
python3 ansible-playbook playbooks/install_start.yaml -i hosts -c ssh -u root
```

## Validation

- Verify playbook completion without failed hosts/tasks.
- Confirm monitoring services/processes are active on target nodes.
- Confirm expected metrics/events are visible in downstream monitoring.

## References

- Proust agent README: <https://github.com/exasol/proust-agent/blob/main/agent/README.md>
- COS monitoring jobs (canonical syntax): `documents/cos/confd-system-and-infrastructure.md`


