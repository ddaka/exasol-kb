---
tool_name: confd_client
doc_type: guide
category: system
title: "SaaS - Amazon EC2 Instance Retirement"
summary: "Operational runbook for AWS EC2 retirement events in SaaS environments, including customer communication and node replacement steps."
---
# SaaS - Amazon EC2 Instance Retirement

## Purpose

Handle AWS retirement notifications where the affected instance must be stopped and started to migrate to healthy hardware.

AWS sends retirement notifications to account administrators. The support flow includes ticket creation, customer communication, controlled downtime, and post-action validation.

## Prerequisites

- Access to the SaaS production AWS account.
- Jira/Exa ticketing access.
- Permission to operate the affected cluster and database.

## Procedure

1. Identify owner and schedule downtime:
- Use [`saas-how-to-get-account-owner`](../troubleshoot/saas-how-to-get-account-owner.md).
- Create support ticket and communicate customer downtime window.

2. Stop customer database using:
- [`how-to-use-confd-client-command-to-interact-with-saas.md`](how-to-use-confd-client-command-to-interact-with-saas.md)

3. Determine impacted node type:
- If retirement affects a **database node**, restart that instance from AWS console, then start and validate database.
- If retirement affects **access node (`n10`)**, stop/start that node in AWS console, then start and validate database.

4. Bring services back and validate:
- Start database (same SaaS confd workflow as above).
- Validate cluster and DB health checks.
- Confirm application connectivity with customer.

## Validation checklist

- Instance state in AWS is `running` on replacement hardware.
- Database status is healthy and accepting connections.
- No critical errors in logs after restart.

## Notes

- Retirement actions are infrastructure-driven; avoid delaying beyond AWS retirement deadline.
- Keep customer communication explicit about expected downtime and recovery confirmation.

## Canonical references

- `documents/cos/confd-system-and-infrastructure.md` (infra lifecycle commands)
- `documents/c4/c4_troubleshooting.md` (general incident handling patterns)
