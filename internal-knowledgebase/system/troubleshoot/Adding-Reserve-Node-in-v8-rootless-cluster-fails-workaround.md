---
tool_name: c4
doc_type: troubleshoot
category: system
title: "Adding Reserve Node in v8 rootless cluster fails"
summary: "The problem applies for all c4 versions lower than:"
---
# Adding Reserve Node in v8 rootless cluster fails

## Problem

The problem applies for all c4 versions lower than:
- 4.24.3
- 4.19.6

When you attempt to add reserve node in v8 rootless cluster, the `c4 host reserve` command fails with an error in the following pattern:

```
exasol@n11:~$ CCC_CONFIG=~/config ./c4 host reserve --ccc-play-rootless true --ccc-host-reserved-addrs 192.168.10.14 --ccc-host-reserved-external-addrs 10.70.11.14 64073762
INFO[2025-04-29 06:59:21] Reserving new nodes for deployment: 64073762-f002-43ae-b91d-548fd5d8a497
ERRO[2025-04-29 06:59:22] node 10.70.11.12: [/usr/local/bin/c4 local reconf --ccc-host-reserved-addrs 192.168.10.14 --ccc-host-reserved-external-addrs 10.70.11.14 --ccc-play-id 64073762 --ccc-play-rootless true]: : non-zero exit code 127
stdout:

stderr:
bash: line 1: /usr/local/bin/c4: No such file or directory
```

## Procedure

As a workaround create a temporary symlink for c4, run this as the root user (or with sudo) on *ALL* exisiting cluster nodes:

```
ln -s /home/exasol/.local/bin/c4 /usr/local/bin/c4
```

Afterwards redo the procedure.

**NOTES:** This article anticipates that the `exasol` user account is used for deploying the rootless database. Make sure `exasol` user exists on the node you are trying to add.

Make sure you ran `c4 _ preplay` on the new node in advance.

You can remove the `/usr/local/bin/c4` symlink after adding the new node, if you don't want to have it.

## Additional References

R&D created a SPOT for this issue to fix it in future:
https://exasol.atlassian.net/browse/SPOT-24855

Online documentation for Add reserve nodes to existing deployment:
https://docs.exasol.com/db/latest/administration/on-premise/nodes/reserve_nodes/add_reserve_nodes_existing.htm
