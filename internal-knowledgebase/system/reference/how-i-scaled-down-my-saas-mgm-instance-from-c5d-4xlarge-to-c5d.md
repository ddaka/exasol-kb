---
tool_name: confd_client
doc_type: reference
category: system
title: "How I scaled down my SaaS mgm instance from c5d.4xlarge to c5d.2xlarge"
summary: "Stop the instance in aws. Change the instance type. Start the instance."
---
# How I scaled down my SaaS mgm instance from c5d.4xlarge to c5d.2xlarge

```
confd_client -c db_stop -a '{db_name: Exasol}'

confd_client -c db_configure -a '{db_name: Exasol,mem_size: 2500MiB,cache_volume_size: 8 GiB}'
```

Stop the instance in aws.
Change the instance type.
Start the instance.

Note: for me it worked with those values. You need to decide for yourself what values you need. I think mem_size: 2GiB is minimum per node (i only had one node)
