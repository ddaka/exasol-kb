---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "Identifying bottlenecks from the Database"
summary: "The rational behind the article is to have documentation and advise to better judge the quality of IO performance in a system. This is required in order to provide feedback when..."
---
# Identifying bottlenecks from the Database

## Extended Problem statement and background information on the reason for this article

The rational behind the article is to have documentation and advise to better judge the quality of
IO performance in a system. This is required in order to provide feedback when troubleshooting and sizing
a system. Currently, the tool **csbench** is being used, which provides throughput values. With only this tool
available, it is hard to give statements about the entire database and/or system performance and to identify bottlenecks
and/or broken hardware. A better understanding of the quality of the current IO performance and entire system
performance is therefor required.

## Identifying bottlenecks from the Database

In general, identifying bottlenecks coming from given usage scenarios is not easily possible. There are many different
factors that influence the systems performance:

1. Is the system in memory?
2. Is storage fragmented?
3. Do we have a broken disk?
4. Is the network between nodes OK?
5. What kind of disks do we have?

If the system is not in memory, there will be lot of IO, however, proper sizing should be considered first. We are an in memory database.
Only if the system is in memory, we should continue looking for IO related problems. This is usually the case, once we have significantly more write than read operations. This can be observed in the PDD Server logs.

### Identifying potential IO problems

In order to identify IO related problems, the first stop is usually the PDD server logs (or EXA Statistics tables), which will give hourly statistics on the read and write throughput, together with the amount of data that has been transferred.
In general, the throughput values in the PDD logs are mostly not in the same range as the values you can achieve by running csbench. Depending on the amount of data, it is perfectly normal that those values are factors 2-4 apart.

### Is storage fragmented?

If storage is fragmented, can be seen if the performance continuously decreases after time passes. This is usually mitigated by a resize of the volume, where the performance is back to normal right after that resizing operation.

### Do we have a broken disk?

Performance regressions that are based on some hardware defects are tricky to detect. In the best case, the SMART tools would give us a number of defect sectors. Another symptom could be that only a single node suffers from performance problems.

### Is the network between nodes OK?

Another reason for decreased IO performance can of course be related to decreased networking performance, as we are performing redundancy operations which require network traffic. This is often not easily reproducible, since the network load might be due to activities on specific time of day.
