---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Is there good known value for high load?"
summary: "We are currently trying to setup a monitoring on basis \"$EXA_MONITOR_DETAILS_LAST_DAY\"."
---
# Is there good known value for high load?

## Question

We are currently trying to setup a monitoring on basis "$EXA_MONITOR_DETAILS_LAST_DAY".

**Is there good known value for high load?**

For example, we could look on the data on da daily basis and look at how often a node had more than 50% of load compared to the average of all nodes.

## Answer

* LOAD measures are somewhat tricky or unreliable. We focus on CPU analysis.
* a CPU imbalance in DETAILS should be tracked down to either CPU_USER (data/processing imbalance) or CPU_DBMS (configuration/hw problem) because this helps to identify issues.
* CPU_USER imbalances (one node doing more) can be easily caused by data distribution or query filtering anomalies (no deeper investigation)
* CPU_USER imbalances could be verified by aggregation on $EXA_PROFILE_DETAILS_LAST_DAY
* in 6.1 there is no CPU_SYS which often gives insights
* rather than a hardware issue, we would check for configuration issues on that node (transparent hugepages, hugepage count/free hugepages)
  * sometimes returned nodes have a bad configuration

Normally we prefer to use CPU values instead of LOAD to check for possible problems. Your criteria for imbalance (one node uses 50% more than average) is ok, make sure the difference between this node and average is large enough (say over 30) and sustained over a longer period of time.

* a CPU imbalance in MONITOR_DETAILS should be tracked down to either CPU_USER (data/processing imbalance) or CPU_DBMS (configuration/hw problem).
* CPU_USER imbalances (one node doing more) can be easily caused by data distribution or query filtering anomalies; in such cases no deeper investigation is needed unless
* CPU_USER imbalances can be verified and confirmed by aggregation on $EXA_PROFILE_DETAILS_LAST_DAY to see where the problem lies
* CPU_DBMS are usually caused by configuration issues on a node rather than a hardware issue; configuration issues include: transparent hugepages, hugepage count/free hugepages, etc. Note that sometimes returned nodes have a bad configuration.
