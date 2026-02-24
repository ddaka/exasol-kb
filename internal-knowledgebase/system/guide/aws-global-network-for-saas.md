---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "AWS Global Network for SaaS"
summary: "Overview of SaaS cross-region networking architecture using AWS Global Network, Transit Gateway, and peering components."
---
# AWS Global Network for SaaS

## Purpose

Explain the AWS networking components used to interconnect SaaS regions and provide troubleshooting orientation.

## Core components

- AWS Global Network (Network Manager container).
- Transit Gateway per supported region.
- Transit Gateway peering between regions.
- Resource shares for cross-account TGW usage.
- VPC peering/attachments for customer DB connectivity.

## Operational model

These resources are typically provisioned automatically by SaaS platform automation.

## Where to inspect in AWS Console

1. Open `VPC` service.
1. Navigate to `Network Manager`.
1. Open `Global Networks`.
1. Inspect attached transit gateways and peering status.

Expected: one attached transit gateway per supported SaaS region.

## Troubleshooting pointers

- Verify TGW attachment state per region.
- Verify peering status between regional TGWs.
- Check route propagation and route-table consistency.
- Use Network Manager monitoring/events where enabled.

## References

- <https://aws.amazon.com/transit-gateway/>
- <https://aws.amazon.com/transit-gateway/network-manager/>
- <https://aws.amazon.com/blogs/networking-and-content-delivery/advanced-troubleshooting-with-aws-transit-gateway-network-manager-route-analyzer/>


