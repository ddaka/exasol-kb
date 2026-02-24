---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "How to adjust proust timeouts"
summary: "This article describes the steps needed to adjust the timeouts for proust checks."
---
# How to adjust proust timeouts

## Overview

This article describes the steps needed to adjust the timeouts for proust checks.

## Diagnosis

The timeouts need to be adjusted after it has been identified that false monitoring alerts are received for a cluster, for example, "Database Connection Issue", even when the DB is correctly showing the latest stats in Grafana (in the cluster state dashboard).

## Explanation

Some clusters/DBs need to have their timeouts adjusted as they might take more time to complete the checks, and therefore, false alerts are generated as the timeout values are too short.

## Recommendation

The steps should be performed in just one node, for example n10 for a V7 cluster or n11 for a V8 cluster, and then the files need to be synchronized to the other nodes.
Note: Before editing, reach out to Marcel or Dren to confirm which specific timeouts/values need to be adjusted.

Example to change the timeout value for the check_session and check_heap.

### STEP 1

Change the timeout values in the proust config
In V7:
vi /etc/proust/proust.config

In V8:
vi /exa/etc/proust/proust.config

Example of the lines where the values are to be checked/edited:
timeout_seconds_check_db: 360
timeout_seconds_check_sessions: 30
timeout_seconds_check_table: 300
timeout_seconds_check_heap: 30

### STEP 2

Sync the file to the other nodes

In V7
cos_sync_files /etc/proust/proust.config

In V8:
cos_sync_files /exa/etc/proust/proust.config

### STEP 3

Change the values in the specific script in /usr/local/bin, in this case, check_session.sh and check_heap.sh:

vi /usr/local/bin/check_session.sh
vi /usr/local/bin/check_heap.sh

Change the timeout value in the cosexec line:

``` bash
cosexec -l "EXAPROD_session" -s -w -r /usr/local/bin/check_sqlquery -query sessions -id "EXAPROD" -timeout "30" | sed 's/^[0-9]*: *//'
```

### STEP 4

Sync the files to the other nodes

cos_sync_files /usr/local/bin/check_session.sh
cos_sync_files /usr/local/bin/check_heap.sh

Note: after updates in a V8 cluster, the values will need to be adjusted again as the monitoring agent has to be installed again.

## Additional References

Exasol Cloud Monitoring [GitHub page](https://github.com/exasol/exasol-cloud-monitoring/blob/main/agent/README.md)


