---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Uninstall pre-NGA v7"
summary: "Exasol pre-NGA v7 is installed in different locations. The installation binaries are usually located under ```/opt``` and there is a systemd unit called ```exasol``` or..."
---
# Uninstall pre-NGA v7

## Problem

Exasol pre-NGA v7 is installed in different locations. The installation binaries are usually located under ```/opt``` and there is a systemd unit called ```exasol``` or ```exasol_<node_id>``` to stop and start the Exasol processes. The installation in ```/opt``` is split in additional folders:
- image
- cl1
- exa
- control

Systemd Unit:
- ```/etc/systemd/system/exasol.service```

CAUTION: If the multiple containers are running on one host there might be multiple systemd units with other names like exasol_n11 or exasol_n12 (check with systemctl status)

In order the completely uninstall the software all those locations need to be cleaned up.

CAUTION: If multple Exasol versions have been installed (e.g. through updates) there might be symlinks under ```/opt/``` linking to different versions.

## Procedure

1. SSH into the server(s) (root or sudo required)
2. Stop the Exasol processes on all nodes
    - ```systemctl stop exasol```
3. Remove Exasol folders under '/opt'
    - ```rm -rf /opt/cl1 /opt/image /opt/exa /opt/control```
4. Remove the systemd unit
    - ```rm -f /etc/systemd/system/exasol.service```
5. Reload systemd
    - ```systemctl daemon-reload```
