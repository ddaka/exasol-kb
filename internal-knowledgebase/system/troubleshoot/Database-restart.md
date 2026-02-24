---
tool_name: cos
doc_type: troubleshoot
category: system
title: "Database restart"
summary: "A database is not responding, EXAoperation is not coming up and you want to restart the cluster."
---
# Database restart

## Overview

A database is not responding, EXAoperation is not coming up and you want to restart the cluster.

## Explanation

For some incidents, you need to restart the cluster. However EXAoperation is not available and the whole system is no responding in a timely manner.
- stop the database
`dwad_client stop-wait <DB_NAME>`
`dwad_client stop-force <DB_NAME>`
- stop EXAStorage
`csctrl -d`
- stop COS or shutdown data nodes
    e.g: `for x in {11..15} ; do ssh n$x -p20 systemctl stop cos ; done`
- reboot license server
`reboot -f`
- after reboot - EXAoperation available again?
- reboot data nodes
- start EXAStorage & database

There are cases where stopping the database and EXAStorage does not work, and even sometimes COS does not stop.
In those cases, you may need to shutdown the data nodes without stopping database and Storage first.

## Additional References

Replace this text with helpful links to other information. Otherwise, delete this section and the Additional References section title.


