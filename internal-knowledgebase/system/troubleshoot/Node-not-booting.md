---
tool_name: cos
doc_type: troubleshoot
category: system
title: "Node Not Booting"
summary: "Physical server was not able to boot up completely."
---
# Node Not Booting

## Problem

Physical server was not able to boot up completely.

## Procedure

1. Check the monitoring logs of **EXAoperation** regarding the node with the issue.

2. SSH into the cluster.

3. Find the IP of the node with the following command:

    ```bash
   psh 'ipmitool lan print | grep "IP Address"'
    ```

   Connect to the iDRAC of the server and select Cold Boot from the power button options to reboot the node.
