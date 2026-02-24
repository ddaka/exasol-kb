---
tool_name: cos
doc_type: troubleshoot
category: system
title: "Private Interface is down"
summary: "The private interface (private0) goes down. The data node is marked as Failed and the database will crash/restart with the Reserve node."
---
# Private Interface is down

## Problem
The private interface (private0) goes down. The data node is marked as Failed and the database will crash/restart with the Reserve node.

## Procedure
1. Inform the customer about this situation and `Deactivate` the node in EXAoperation.
2. Connect to the data node via the public IP (public0) and try to restart the `private0` interface.
```
ssh {public_IP} -p20
ip link set dev private0 up
```
3. Check if the link of `private0` is still down.
```
ip link
```
4. If the link is still down, the `COS` must be stopped as it will flood the Appserver logs and cause the `/` root filesystem to get full.
```
systemctl stop cos
```

When the `private0` link is back online (UP), one can re-activate the node in DB's configuration and start the cluster services (COS) from EXAoperation.

## Additional References
[Replace Active Node with Reserve Node](https://docs.exasol.com/db/7.1/administration/on-premise/nodes/replace_active_node.htm?)
