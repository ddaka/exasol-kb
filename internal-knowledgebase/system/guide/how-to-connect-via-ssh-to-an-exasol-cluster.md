---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "Connect to an Exasol Cluster via SSH"
summary: "Set up SSH key-based root access to an Exasol cluster and synchronize authorized keys across nodes."
---

# Connect to an Exasol Cluster via SSH

## Overview

This guide describes how to enable SSH key-based access to an Exasol cluster (root user) and propagate authorized keys across nodes.

## Prerequisites

- SSH client tools (`ssh`, `ssh-keygen`, `ssh-copy-id`) on your local system.
- Temporary password-based access to a cluster management node.
- Support workflow alignment if your environment requires Exasol Support for root access handling.

## Procedure

### 1. Generate an SSH key pair

```bash
ssh-keygen
```

Default output files:

- Private key: `~/.ssh/id_rsa`
- Public key: `~/.ssh/id_rsa.pub`

### 2. Add public key to the cluster management node

```bash
ssh-copy-id root@DESTINATION_HOST
```

Provide the root password when prompted.

### 3. Log in to the cluster

```bash
ssh root@DESTINATION_HOST
```

### 4. Synchronize authorized keys to all nodes

```bash
cos_sync_files ~/.ssh/authorized_keys
```

### 5. Validate access

- Exit and reconnect.
- Test SSH login to additional cluster nodes.

## Security Notes

- Keep private keys secure and never share them.
- Share only the public key (`id_rsa.pub`).
- Use passphrase-protected keys where possible.

## References

- <https://www.openssh.com>
- <https://www.ssh.com/academy/ssh/keygen>
- <https://learn.microsoft.com/windows-server/administration/openssh/openssh_keymanagement>
