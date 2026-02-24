---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Dead Telegraf Agent"
summary: "An alert was created as follows:"
---
# Dead Telegraf Agent

## Problem

An alert was created as follows:

Case Subject: Database Failsafety

Case Description:

- Check: Dead Telegraf Agent is: crit

## Procedure

This alerts shows us that the telegraf agent is not running on the node. On Tags field, we can find Cluster ID, Account Group, Database Name and node id effected.

Action:

1) Check the state of the telegraf agent in Grafana
2) If the state of telegraf agent is still down, wait for some minutes, as it could recover by itself. It happens to fails temporarily due to network interruption
3) If this is still down, restart the telegraf agent by running the below commands:

* Find the proust_partition_id with cosps -n
* `cosrm -a <proust_partition_id>`
* `cosexec -l proust -s -a --auto-add --auto-restart /usr/bin/telegraf -pidfile /var/run/telegraf/telegraf.pid -config /etc/telegraf/telegraf.conf -config-directory /etc/telegraf/telegraf.d`

You can find the cosexec also in /etc/rc.local_cos

If this does not work restart it with:

```
python3 ansible-playbook playbooks/stop.yaml -i hosts -c ssh -u root
python3 ansible-playbook playbooks/start.yaml -i hosts -c ssh -u root
```

## Additional References


