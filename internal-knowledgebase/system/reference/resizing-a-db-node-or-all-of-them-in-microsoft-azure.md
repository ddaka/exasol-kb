---
tool_name: internal-knowledgebase
doc_type: reference
category: system
title: "Resizing a DB node (or all of them) in Microsoft Azure"
summary: "to check valid instances:"
---
# Resizing a DB node (or all of them) in Microsoft Azure

1. On EXAOperation as "Admin", stop all DBs
2. then stop Storage Services
3. Suspend all database nodes (assuming you are about to resize all nodes)
4. Change the instance type:
to check valid instances:

```
$ az vm list-sizes --location <your_location>
```

to resize the node:

```
$ az vm resize --size <new_instance_type> -n <node> -g <resource_group>
```

or all of them (the example below is for 3 nodes):

```
$ for i in 1 2 3; do az vm resize --size <new_instance_type> -n <node_name - last_digit>$i -g <resource_group>; done
```

5. Once all the nodes have been resized (reboot is needed), we need to:
     5.1 resume the nodes,
     5.2 start storage services
     5.3 Edit the database to adjust the parameter "Database RAM (GiB)"
     5.4 start the database
