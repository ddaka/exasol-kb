---
tool_name: internal-knowledgebase
doc_type: troubleshoot
category: system
title: "IMPORT slowdown after migration to v8 or OS update"
summary: "Starting with Exasol v8, customers can choose their own Linux OS. In certain deployments (for example, on Ubuntu 22.04.5 with kernel 5.15.0-153) IMPORT statements (and potentially..."
---
# IMPORT slowdown after migration to v8 or OS update

## Problem

Starting with Exasol v8, customers can choose their own Linux OS. In certain deployments (for example, on Ubuntu 22.04.5 with kernel 5.15.0-153) IMPORT statements (and potentially other statements, such as SELECT and COMMIT) may become significantly slower (more than 2x) under rare circumstances.
The reason: the kernel enabled expensive Spectre v2 CPU security mitigations because the hardware did not provide updated microcode.

## Explanation

The OS and kernel updates may enable security mitigation against Spectre v2 and other risks.
Those software mitigations may cause measurable performance issues in the database - increased resource usage like CPU and NET but also pressure on the i/o system.
Especially IMPORT (but also SELECT and COMMIT) statements' performance could be impacted.

In very rare cases security mitigations can have such effects. Though, in our research and testing we discovered that especially software mitigations executed by the kernel cost a lot of performance.
But those mitigations are rather a fallback if hardware mitigations (new microcode) are unavailable. Hardware mitigations come with Firmware, IDRAC and BIOS updates.
One can detect that software mitigations kick in by checking for the value `ibpb_no_ret` showing up in `/proc/cpuinfo` (bugs line).

## Solutions

### . Update All Hardware Components

Apply newest BIOS, firmware, IDRAC, and microcode on all cluster nodes. This usually allows the kernel to use hardware mitigations, which have <5% performance cost.

### . Evaluate Kernel Mitigation Options

You may consider lowering or even temporarily disabling some of the mitigations to improve performance. Proceed with caution.
Disabling the mitigations entirely can be useful for confirming whether they are indeed the source of the performance issue, but it is strongly not recommeneded to run the database in production with all mitigations turned off.

Test different protection levels:

- Safe but still fast: `spectre_v2_user=prctl spec_store_bypass_disable=off`
- Higher risk / higher performance: `spectre_v2_user=off spec_store_bypass_disable=off`
- Not recommended except for isolated systems or to prove the root cause: `mitigations=off`

### . Roll Back to a Previous Kernel

Use the old kernel with updated OS and wait until a better/faster kernel becomes available.

### . Improve overall ETL performance

If the solutions above cannot be implemented, you may still choose to use the latest kernel and OS and accept the associated performance impact.
Some particularly long IMPORT operations can often be optimized or parallelized to speed up ETL workloads.
One approach is to use multiple source statements within a single IMPORT to parallelize the connections. This method typically scales well and keeps complexity manageable.

An example rewrite of an IMPORT of a table test.T1  might look like the following

```sql
import into test.target from jdbc at 'jdbc:exa:<REMOTE_DB_HOST>:<PORT>' user '<REMOTE_USER>' identified by '<REMOTE_PASS>'
STATEMENT 'SELECT * FROM test.t1 WHERE MOD(rowid, 4) = 0'
STATEMENT 'SELECT * FROM test.t1 WHERE MOD(rowid, 4) = 1'
STATEMENT 'SELECT * FROM test.t1 WHERE MOD(rowid, 4) = 2'
STATEMENT 'SELECT * FROM test.t1 WHERE MOD(rowid, 4) = 3';
```
