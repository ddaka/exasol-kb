---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "Deploy Exasol on Docker on a machine/VM with multiple NUMA nodes"
summary: "Deploy Exasol Docker containers with NUMA-aware CPU pinning and EXAConf tuning on large hosts."
---
# Deploy Exasol on Docker on a machine/VM with multiple NUMA nodes

## Purpose

Run Exasol on large Docker hosts with multiple NUMA nodes while minimizing cross-node memory access penalties.

## When to use this

Use this approach when the host has multiple NUMA nodes and you want to pin Exasol containers to NUMA-local CPU sets.

## Prerequisites

- SSH access to the host.
- Docker installed and running.
- Exasol Docker image pulled (for example `exasol/docker-db:<version>`).
- Planned node IDs, memory split, and storage mount paths.

## 1) Identify NUMA node to CPU mapping

```shell
lscpu | grep node
numactl -H
```

Example interpretation:

- NUMA node 0: CPUs `0-39`
- NUMA node 1: CPUs `40-79`

In this example, run two containers and pin each to one NUMA node.

## 2) Prepare container data directories

```shell
mkdir -p /var/mnt/exasol/n{11..12}
```

Generate initial template data:

```shell
docker run -v /var/mnt/exasol/n11:/exa --rm -i exasol/docker-db:<version> init-sc --template --num-nodes 2
```

Clone template content to the second node directory:

```shell
cp -r /var/mnt/exasol/n11/* /var/mnt/exasol/n12/
```

## 3) Update EXAConf for NUMA-aware sizing

Review storage/network settings and configure database node list. In `[DB : DB1]`, include NUMA-related DB params:

```ini
[DB : DB1]
   Version = 7.0.7
   MemSize = 576 GiB
   Port = 8563
   Owner = 500 : 500
   Nodes = 11,12
   NumActiveNodes = 2
   DataVolume = DataVolume1
   Params = -useOptimizedAllocationForNuma=0 -nrOfCores=40
```

Optional custom bridge network example:

```shell
docker network create --subnet 10.0.0.0/24 --gateway 10.0.0.1 --driver bridge exasol_net
```

## 4) Build CPU lists for `--cpuset-cpus`

```shell
for i in {0..39}; do echo -n "$i,"; done
for i in {40..79}; do echo -n "$i,"; done
```

## 5) Start NUMA-pinned containers

```shell
docker run --name n11 --cpuset-cpus="0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39" --shm-size=2g --detach --memory=280g --memory-swap=0g --memory-reservation=280g -p <ip_address>:8563:8563 -p <ip_address>:2580:2580 -p <ip_address>:443:443 --network=bridge --privileged -v /var/mnt/exasol/n11:/exa exasol/docker-db:7.0.7 init-sc --node-id 11

docker run --name n12 --cpuset-cpus="40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,55,56,57,58,59,60,61,62,63,64,65,66,67,68,69,70,71,72,73,74,75,76,77,78,79" --shm-size=2g --detach --memory=280g --memory-swap=0g --memory-reservation=280g --network=bridge --privileged -v /var/mnt/exasol/n12:/exa exasol/docker-db:7.0.7 init-sc --node-id 12
```

Ports used:

- `8563`: database
- `2580`: BucketFS
- `443`: ConfD/XML-RPC

## 6) Validate startup

```shell
docker logs -f n11
```

Also validate container health, DB startup, and expected NUMA/CPU pinning.


