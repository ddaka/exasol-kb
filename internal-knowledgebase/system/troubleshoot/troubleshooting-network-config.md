---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Troubleshooting network configuration when the database connection is up and online, but customer still can't connect"
summary: "In some cases, the database appears to be up and running, and a connection using `exaplus` from the localhost works without issues. However, the customer is still unable to..."
---
# Troubleshooting network configuration when the database connection is up and online, but customer still can't connect

## Problem

In some cases, the database appears to be up and running, and a connection using `exaplus` from the localhost works without issues. However, the customer is still unable to connect remotely.

This issue is likely related to the server's network configuration. It’s important to verify the following:

- The network interfaces are configured correctly.
- No duplicate IP addresses are assigned to multiple interfaces or bonds.
- A default route is set on each node.

Customers experiencing this issue should follow the steps below to confirm whether they're affected.

## Procedure

Follow these steps to troubleshoot and resolve the issue:

1. **Test local database connection**

Run the following command from the localhost to verify that the database itself is functional:

   ```bash

   exaplus -c <connection_string> -u <user> -p <password>
   ```
   If this works, proceed to check the network configuration.

2. **Check network interfaces**

   Run:

   ```bash

   ip addr

   ```
   - Ensure all required interfaces are up and have the correct IP addresses.
   - Make sure no IP address is assigned to more than one interface or bond.
   If a duplicate IP assignment is found, remove the incorrect one:

   ```bash

   ip addr del <wrong_ip>/24 dev <interface>

   ```
   3. **Check default route**

   Each node must have a valid default route configured. You can check this with:

   ```bash

   ip route

   ```
   If the default route is missing, add it using:

   ```bash

   ip route add default via <gateway_ip> dev <interface>

   ```

After completing these steps, test the remote connection again. If problems persist, further investigation into firewall settings, VLAN configurations, or routing tables may be required.
