---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "How to set up Guacamole"
summary: "Request workflow for provisioning a Guacamole support host to access customer systems."
---
# How to set up Guacamole

## Purpose

Request a Guacamole support host for controlled access to customer nodes.

## Prerequisite

Ensure customer VPN access request is completed first:

- `how-to-setup-customer-VPN.md`

## Request workflow

1. Collect customer node IP addresses and hostnames.
2. Create Service Desk ticket with:
   - Category path: `Request / Software / General Request / Support Environment`
   - Summary: `Setup Guacamole for customer <customer_name>`
   - Description:
     - Node IP addresses and hostnames
     - Requested support-host name (aligned with Salesforce account name)

## Validation

- Ticket accepted by responsible team.
- Support host provisioned and reachable.


