---
tool_name: cos
doc_type: troubleshoot
category: system
title: "INTERNAL: Disable martian packet kernel warnings"
summary: "Some IT security compliance standards (PCI-DSS) require the logging of martian packets (packets that arrive from an unexpected network interface). This is done to allow detection..."
---
# INTERNAL: Disable martian packet kernel warnings

## Overview

Some IT security compliance standards (PCI-DSS) require the logging of martian packets (packets that arrive from an unexpected network interface). This is done to allow detection of spoofed packets. We implement this by default to comply with these standards.

Some customer's network setups on the other hand generate these packets constantly and can't or won't be changed(fixed?). This creates many spurious warnings and drowns out useful log messages.

This article describes a way to permanently disable these messages on a legacy EXAoperation installation if needed.

## Prerequisites

Check that the file */etc/rc.local.cos* is either identical on all of the nodes or does not exist on any:

```text
cluster1 [root@n0010 ~]# [[ -e /etc/rc.local.cos ]] && sha256sum /etc/rc.local.cos | psh 'sha256sum -c 2>&1'
0011: /etc/rc.local.cos: OK
0012: /etc/rc.local.cos: FAILED
0012: sha256sum: WARNING: 1 computed checksum did NOT match
0013: sha256sum: /etc/rc.local.cos: No such file or directory
0013: /etc/rc.local.cos: FAILED open or read
0013: sha256sum: WARNING: 1 listed file could not be read
```

```text
cluster1 [root@n0010 ~]# [[ ! -e /etc/rc.local.cos ]] && psh '[[ ! -e /etc/rc.local.cos ]] || echo "ERROR, file exists"'
0011: ERROR, file exists
0014: ERROR, file exists
```

Otherwise, check the contents of each file and talk with whoever put them there.

## How to permanently disable the warnings

1. Add the following to */etc/rc.local.cos* on the license server:

    ```shell
    sysctl -e net.ipv4.conf.all.log_martians=0
    sysctl -e net.ipv4.conf.default.log_martians=0​
    ```

2. Synchronize the file to all nodes:

    ```shell
    cos_sync_files /etc/rc.local.cos
    ```

3. Apply the settings to the running system

    ```shell
    cosexec -a sysctl -e net.ipv4.conf.all.log_martians=0
    cosexec -a sysctl -e net.ipv4.conf.default.log_martians=0​
    ```
