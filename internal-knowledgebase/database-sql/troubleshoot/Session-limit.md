---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: database-sql
title: "Active session limit reached"
summary: "Response procedure for alerts indicating active session limits are near or at capacity."
---
# Active session limit reached

## Problem

Monitoring raises one of the following alerts:

- `Query queue limit of active sessions nearly reached`
- `Limit of active sessions has been reached`

This condition can cause degraded performance and blocked workload execution.

## Procedure

1. Confirm the affected database and current session pressure.
1. Notify the customer that active session capacity is exhausted or close to exhaustion.
1. Ask the customer to terminate idle sessions with open transactions and reduce concurrent workload.
1. Share documentation link: ["Active Session Limit" Reached](https://exasol.my.site.com/s/article/Active-Session-Limit-Reached?language=en_US).
1. Check whether a customer support case already exists for reduced performance and align updates.

## Customer Message Template

```text
Dear <CUSTOMER NAME>,

Our monitoring detected that the active session limit for <DATABASE NAME> is nearly reached or has been reached.
Please take action to reduce active session usage (for example, terminate idle sessions with open transactions).
For details, see:
"Active Session Limit" Reached
https://exasol.my.site.com/s/article/Active-Session-Limit-Reached?language=en_US

If you need assistance, please let us know.

Thank you and best regards,
<YOUR NAME>
```

## Result

The incident is resolved when active session usage returns below limit and workload performance stabilizes.

---

_We welcome feedback on this troubleshooting article._
