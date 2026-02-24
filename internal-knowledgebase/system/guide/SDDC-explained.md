---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "SDDC explained"
summary: "Architecture and setup guidance for Synchronous Dual Data Center (SDDC) in Exasol v7/v8."
---
# SDDC explained

## Scope

This guide explains Synchronous Dual Data Center (SDDC) architecture and setup principles for Exasol v7 and v8.

## What SDDC means

SDDC (also called stretched storage) distributes storage roles across two data centers:

- `SITE_A` hosts master segments.
- `SITE_B` hosts redundant segments.

Two database instances share the same data volume, but only one instance runs at a time.

## Core design rules

- Use equal node counts across `SITE_A` and `SITE_B`.
- Configure storage volumes with redundancy `2`.
- Add nodes in strict order: all `SITE_A` nodes first, then `SITE_B` nodes.
- Keep one primary database instance for `SITE_A` and one DR instance for `SITE_B`.

Incorrect node order can assign master/redundant segments unexpectedly and complicate operations.

## Setup sequence

1. Install and activate all cluster nodes.
1. Create data/archive volumes with redundancy `2`.
1. Add `SITE_A` and `SITE_B` nodes to volumes in correct order.
1. Create two database instances that share the same data volume.
1. Assign `SITE_A` nodes to the primary DB and `SITE_B` nodes to the DR DB.
1. Start and stop each database at least once to clear initialization flags.
1. Leave only one side online during normal operation.

## Operational behavior

- When running from `SITE_B` during a `SITE_A` outage, degraded-volume conditions are expected.
- Segment roles may shift (for example redundant segments acting in deputy role) until recovery.
- After `SITE_A` returns, segments resynchronize and normal role distribution is restored.

## Failover and fallback

- Failover: stop `SITE_A` database (if reachable), start `SITE_B` database.
- Fallback: stop `SITE_B` database, start `SITE_A` database after recovery.

Always validate segment health and synchronization state after each switch.

## References

- <https://docs.exasol.com/db/latest/planning/fail_safety.htm>
- <https://docs.exasol.com/db/7.1/planning/business_continuity/sddc_details.htm>


