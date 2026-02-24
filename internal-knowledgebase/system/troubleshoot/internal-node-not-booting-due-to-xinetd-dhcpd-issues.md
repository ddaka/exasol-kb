---
tool_name: cos
doc_type: troubleshoot
category: system
title: "Node not booting because of xinetd/dhcpd running on data nodes"
summary: "The xinetd and dhcpd are mandatory for booting up nodes. These daemons must ONLY run on the MGMT node."
---
# Node not booting because of xinetd/dhcpd running on data nodes

## Description

The xinetd and dhcpd are mandatory for booting up nodes. These daemons must ONLY run on the MGMT node.

Sometimes it can happen e.g. when EXAoperation was running on a data node, that these daemons will remain running on that data node. If this is the case, data nodes can not boot up properly.

## Diagnosis

You can check if these daemons are running on data nodes with the following command:

psh 'ssh localhost -p20 ps fax | grep -E 'dhcpd|xinetd''

## How to fix

1. Remove EXAoperation COS partition:

      cosrm -a appserverd-ID

2. Kill all xinetd and dhcpd processes also on MGMT node

3. Restart EXAoperation on MGMT node using the following command:

      cosexec --single-instance --auto-restart --auto-add -- $COS_DIRECTORY/libexec/appserverd


