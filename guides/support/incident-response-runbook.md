---
title: Exasol Incident Response Runbook (Quick Reference)
description: SLA-driven runbook for customer calls, Nagios alerts, triage, log collection, and common remediation steps.
tool_name: general
doc_type: troubleshooting
category: Incident Response
subcommands:
  - psh
  - dwad_client
  - cosps
  - csinfo
  - csrec
  - logd_collect
  - get_support_info
---

# Exasol Incident Response Runbook (Quick Reference)

This document consolidates the incident response and operations notes into a practical troubleshooting guide.

## 1) Response SLAs

- Respond to customer calls or Nagios alerts within 1 hour.
- Create a ticket and inform the customer in the first response window.
- Send updates at least 1 hour after the last customer reaction.
- If hotline option `1` was pressed, T&Cs are accepted; support starts immediately and charging is handled afterward.

## 2) General Incident Rules

- Always post a short status update, even when investigation is still in progress.
- Record phone calls in JIRA as call notes.
- Check the customer service package; involve SE or Sales when scope/coverage is unclear.
- If two blocker incidents occur in parallel, prioritize by customer segment and activate on-call backup.
- Escalate to development during business hours if unresolved after 1 hour.
- Typical rates: business hours `EUR 150/h`, out-of-hours `EUR 225/h`.

## 3) Call Handling

### Customer Calls

- Confirm caller identity and affected DB/cluster.
- Ask for symptom details, start time, attempted steps, business priority, and session IDs.
- Verify VPN/remote access and arrange screen sharing within the next hour.

### Nagios Alerts

- Acknowledge call/alert (for hotline workflows: press `3` to clear message queue).
- Check SMS/email context and existing tickets.
- Create support + EXA tickets and notify customer.
- For required changes, raise a change request and obtain approval first.

## 4) Quick Triage Checklist

1. Check EXAoperation web UI health.
2. From SSH on cluster, run:
   - `psh uptime`
   - `psh ping -c1 license`
   - `dwad_client list`
3. Inspect cluster process/storage state:
   - `cosps -r` or `cosps -N`
   - `csinfo -v`
   - `csrec -l`
4. Check capacity: `psh df -h`
5. Inspect kernel/system messages on suspect nodes: `dmesg`
6. Add temporary monitoring downtime if needed to prevent alert storms.

## 5) Key Logs And Commands

### Management Node Logs

- EXAoperation logs:
  - `/usr/opt/EXASuite-6/<COS-Version>/var/exaoperation/log/*.log`
- OS logs:
  - `/var/log/boot.log`
  - `/var/log/all.log`

### Client Node (initrd) Logs

- `/var/log/hddmount.log`
- `/var/log/hddinit.log`
- `/var/log/cos_startup.log`

### Database And COS Logs

- DB process logs:
  - `/d02_data/{database name}/log/process/*`
- COS logs:
  - `/var/log/cored/*`
  - `/var/log/logd/*`

### Operational Commands

- `logd_collect <Service>`: collect and time-sort logs across nodes
- `dwad_client list`: list DB state and recent events
- `psh df -h`: detect full filesystems
- `get_support_info -b <level>`: create support bundle (`1-4`)
- `cos_mkbootimg`: rebuild boot images

## 6) Common Incident Playbooks

### Cluster Restart Fails

1. Review management boot/logs:
   - `/var/log/boot.log`
   - `/var/log/logd/Appserverd.log`
2. For data node boot issues, connect with `rssh nX`.
3. Inspect:
   - `/var/log/exaopnodestart`
   - `dmesg`

### Backup Stuck Or Failed

1. Check pddserver and backup-related logs via `logd_collect`.
2. Use:
   - `dwad_client abort-backup <id>`
   - `psh sdfs list <vol-ID>`
3. Validate archive/storage free space.
4. Consider DB restart and/or incremental backup strategy if needed.

### Database Hangs Or Does Not Start

1. Check ConnectionServer logs and session/concurrency limits.
2. Capture diagnostics: `get_support_info -b 1..4`
3. Controlled restart:
   - `dwad_client stop-wait <db>`
   - `dwad_client start-wait <db>`

### Hardware Failure

1. Inspect hardware event logs:
   - `ipmitool sel elist`
   - vendor tools such as `megacli`, `omreport`, `hplog`
2. For node maintenance:
   - stop DB
   - deactivate affected node
   - start DB
3. Coordinate hardware replacement if required.

## 7) Planning Guidance

- For planned operational tasks, add a minimum 25 percent buffer to estimated duration.
- Common tasks include migration, backup/restore migration, update/upgrade, security patching, IP moves, and cluster expansion.

## 8) Notes

- Command availability and behavior can vary by Exasol/COS version; verify syntax in your target environment.
- Use change management for disruptive actions in production.

## Incident Calendar Placeholders (2026)

- [January 2026](incident-calendar/2026/2026-01-january.md)
- [February 2026](incident-calendar/2026/2026-02-february.md)
- [March 2026](incident-calendar/2026/2026-03-march.md)
- [April 2026](incident-calendar/2026/2026-04-april.md)
- [May 2026](incident-calendar/2026/2026-05-may.md)
- [June 2026](incident-calendar/2026/2026-06-june.md)
- [July 2026](incident-calendar/2026/2026-07-july.md)
- [August 2026](incident-calendar/2026/2026-08-august.md)
- [September 2026](incident-calendar/2026/2026-09-september.md)
- [October 2026](incident-calendar/2026/2026-10-october.md)
- [November 2026](incident-calendar/2026/2026-11-november.md)
- [December 2026](incident-calendar/2026/2026-12-december.md)
