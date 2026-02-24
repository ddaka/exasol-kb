---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "SDDC cluster interconnect: requirements and bandwidth planning"
summary: "Estimate and validate minimum inter-site bandwidth for SDDC clusters based on active-node disk throughput and concurrent backup load."
---
# SDDC cluster interconnect: requirements and bandwidth planning

## Purpose

Size the inter-site network link for SDDC deployments to avoid replication and commit bottlenecks.

## Key factors

- Active node count
- Disks per node
- Sustained disk write throughput per disk
- Concurrent backup throughput
- Latency, encryption/firewall overhead, and shared-link contention

## Baseline formula

```text
Required bandwidth (MB/s) = active DB nodes * disks per node * disk throughput per disk (MB/s)
```

If backups run concurrently, add aggregate backup throughput.

## Example

- Commit load: `3 nodes * 4 disks * 200 MB/s = 2400 MB/s`
- Backup load: `3 nodes * 250 MB/s = 750 MB/s`
- Total: `3150 MB/s`
- Convert to Gbit/s: `3150 / 125 = 25.2 Gbit/s`

Recommended minimum in this example: **25 Gbit/s**.

## Throughput reference

| Interconnect | Throughput | 30 GB transfer | 3 GB transfer |
| --- | --- | --- | --- |
| 1 Gbit/s | 125 MB/s | ~245.8 s | ~24.6 s |
| 10 Gbit/s | 1250 MB/s | ~24.6 s | ~2.5 s |
| 20 Gbit/s | 2500 MB/s | ~12.3 s | ~1.2 s |
| 40 Gbit/s | 5000 MB/s | ~6.1 s | ~0.6 s |

## Recommendations

- Reserve dedicated bandwidth for SDDC traffic.
- Include failover and backup peaks in capacity planning.
- Validate assumptions with real workload tests.
- Add safety margin above theoretical minimums.


