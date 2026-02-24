---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "INTERNAL: Cloud UI - disk enlargement - Volume Modification Size Limit Exceed"
summary: "The default volume modification for AWS is **100 TiB**..."
---
# INTERNAL: Cloud UI - disk enlargement - Volume Modification Size Limit Exceed

## Explanation

The default volume modification for AWS is **100 TiB** ([https://docs.aws.amazon.com/general/latest/gr/aws_service_limits.html](https://docs.aws.amazon.com/general/latest/gr/aws_service_limits.html "Follow")). If the customer has an enterprise AWS support they can request and increase this limit.

If the customer is not aware of this limit and adds disk space to a cluster using the Cloud UI, they will receive an error about the volume modification size limit exceeded. Workaround to solve the issue:

* Stop DB
* Stop Storage
* Stop nodes
* Check volume sizes on AWS console
* Wait for Optimisation for increased volumes to finish
* Modify the volume sizes for remaining volumes manually
* Start nodes
* Start storage
* Enlarge disks for all nodes manually
* Start database
