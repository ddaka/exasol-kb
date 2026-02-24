---
tool_name: cos
doc_type: troubleshoot
category: system
title: "How-to start EXAsolution without the management node (N10)"
summary: "One of our customer's management nodes is failing due to hardware issues and we were asked to bring back online the database without N10 in the equation."
---
# How-to start EXAsolution without the management node (N10)

## Overview

One of our customer's management nodes is failing due to hardware issues and we were asked to bring back online the database without N10 in the equation.

## Prerequisites

The data nodes are up and running and the database, storage, and cluster services were gracefully stopped.

## How to start EXAsolution without the management node (N10)

1. Connect to the data nodes via port 20 and start COS services `systemctl start cos`
2. After running `cosps -N` you should see all nodes in Online state
3. Export variable `SETUP_DIR="$COS_DIRECTORY/var/setup"`
4. Start all Exasol services (source file `/etc/cos/cluster_services`)

```basic
    cosexec --single-instance --auto-add --auto-restart -- \
	    $COS_DIRECTORY/libexec/logd \
	    --default-log-dir /var/log/logd
    cosexec --single-instance --auto-add --auto-restart -- \
	    $COS_DIRECTORY/libexec/lockd
    cosexec --single-instance --auto-add --auto-restart -- \
	    $COS_DIRECTORY/libexec/dwad \
	    --backupfile $SETUP_DIR/dwad.dump \
	    --store-config $SETUP_DIR/dwad.dump \
	    --store-interval 10
    cosexec --single-instance --auto-restart --auto-add -- \
	    $COS_DIRECTORY/libexec/appserverd
```

5. Now you should be able to access EXAopeartion via one of the IP addresses of the data nodes

6. Start EXAstorage

7. Start EXAsolution

## Additional Notes

The data nodes cannot be rebooted as they will not be able to boot over PXE if the management node is not accessible.

## Additional References

[Start/Stop a Database - On Premise | Exasol Documentation](https://docs.exasol.com/administration/on-premise/manage_database/start_stop_db.htm)

[Stop and Start Storage Services - On Premise | Exasol Documentation](https://docs.exasol.com/administration/on-premise/manage_storage/stop_start_storage_service.htm)
