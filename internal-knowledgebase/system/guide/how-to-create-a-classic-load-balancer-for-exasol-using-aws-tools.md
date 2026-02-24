---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "How to create a Classic Load Balancer for Exasol using AWS tools"
summary: "Create an internet-facing AWS Classic Load Balancer for Exasol TCP connections on port 8563 and validate node health behavior."
---
# How to create a Classic Load Balancer for Exasol using AWS tools

## Purpose

Expose Exasol database connectivity through a single public DNS endpoint using AWS Classic Load Balancer (CLB).

## Important notes

- Prefer encrypted DB connections (`-forceProtocolEncryption=1`).
- Review CLB pricing before deployment.
- `EXA <-> EXA` `IMPORT/EXPORT` is not supported through this pattern.
- This guide assumes a cluster with active and reserve nodes in private subnets.

## Prerequisites

- AWS account permissions for ELB, EC2, VPC, and Security Groups.
- Exasol cluster and subnet details.
- Decision on public access scope and security controls.

## 1) Create Classic Load Balancer

In AWS Console:

1. Open `EC2` -> `Load Balancers`.
2. Create a new **Classic Load Balancer**.
3. Select the target Exasol VPC.
4. Configure listener:
   - Load Balancer protocol: `TCP`
   - Load Balancer port: `8563`
   - Instance protocol: `TCP`
   - Instance port: `8563`
5. Select required subnet(s).

## 2) Configure security group

Allow inbound traffic to CLB listener port `8563` from approved source ranges only.

## 3) Configure health check

Set health check to TCP on port `8563`.

Recommended:

- Interval: `>=30s` (avoid aggressive probing)

## 4) Register instances

- Register all relevant Exasol DB nodes (active + reserve per operational design).
- Disable features not required by your architecture (for example cross-zone load balancing/connection draining) according to environment standards.

## 5) Finalize and validate

- Create the load balancer and wait for DNS propagation.
- Verify all registered nodes report healthy.
- Test DB connectivity through CLB DNS endpoint on `8563`.
- Confirm failover behavior during node role changes.

## References

- CLB pricing: <https://aws.amazon.com/elasticloadbalancing/classicloadbalancer/pricing>
- Alternative architecture (HAProxy + floating IP): <https://exasol.my.site.com/s/article/How-to-create-a-HAproxy-Load-Balancer-with-floating-IP>


