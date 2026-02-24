---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "SaaS - High Swap Usage"
summary: "If there is a high swap usage it means that either the memory is fully utilized or there is a configuration issue. High Swap Usage means that kernel is forced to continuously move..."
---
# SaaS - High Swap Usage

## Overview

If there is a high swap usage it means that either the memory is fully utilized or there is a configuration issue. High Swap Usage means that kernel is forced to continuously move memory pages to swap and back to RAM to keep application running.

## How to fix high swap usage

If there is a memory leak, the only way of permanent fixing swap usage is to fix the bug which causes the memory leak. However, one can mitigate the issue at least until the permanent issue is available.

### Step 1

Check swapiness value on the system:

```bash
$ cat /proc/sys/vm/swappiness
1
```

Our default swapiness value is 1, i.e. it means minimum amount of swapping without disabling it entirely. For some reason, if swappiness setting has high value then aggressive swapping is expected. But for our application this is not desired and it should be changed to the low value if it is the case.

### Step 2

Check memory and swap usage using free command:

```bash
$ free -h
              total        used        free      shared  buff/cache   available
Mem:           62Gi        27Gi        32Gi       950Mi       1.5Gi        31Gi
Swap:           9Gi       149Mi       9.9Gi
```

### Step 3

The only way to bring swap usage down is stop database manually. One can stop database using `dwad_client`. After stopping the database wait and check swap usage if it is dropped. If the swap usage is still very high, clear the swap usag by disabling and enabling the swap.

Disable swap:

```bash
$ swapoff -a
```

Wait for approximately 30 second to give the operation time to complete and enable swap again:

```bash
$ swapon -a
```

### Step 4

Inform the customer and collect logs following [this runbook](how-to-get-log-files-from-exasol-saas-systems-temporary.md) and assign the support to development to investigate the root cause of the high swap usage.
