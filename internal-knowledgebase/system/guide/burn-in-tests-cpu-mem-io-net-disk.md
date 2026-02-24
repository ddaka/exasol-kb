---
tool_name: cos
doc_type: guide
category: system
title: "Burn-in tests (CPU, MEM, IO, NET, DISK)"
summary: "Hardware certification and stress-test runbook for Exasol clusters across compute, storage, and network dimensions."
---
# Burn-in tests (CPU, MEM, IO, NET, DISK)

## Purpose

Validate hardware stability and performance before production use.

## Test groups

- Functional checks (installation, hardware/log sanity, patch state).
- Stress checks (CPU, memory, disk, network under sustained load).
- Performance checks (throughput/latency benchmarks).

## Prerequisites

- Hardware meets Exasol minimum/certification requirements.
- Test window approved.
- Required tooling available (`csbench`, `dd`, `iostat`, `iotop`, `iperf`, etc.).

## Recommended workflow

### 1. Functional validation

- Verify cluster install and node health.
- Validate storage/network visibility.
- Check logs for hardware-level faults.

### 2. Storage and IO benchmark

- Use dedicated temporary test volumes.
- Run `csbench` with multiple block sizes and patterns.
- Monitor with `iotop`/`iostat`.

### 3. Network benchmark

- Use `iperf` for throughput and parallel-stream checks.
- Validate expected MTU profile and cross-node bandwidth.

### 4. CPU and memory stress

- Use stress tooling with bounded, repeatable parameters.
- Observe throttling, instability, and error behavior.

### 5. Consolidate results

- Compare against expected baseline for instance/hardware class.
- Flag bottlenecks (IO wait, skewed bandwidth, thermal throttling, etc.).

## Cautions

- Temporary benchmark volumes are not for production DB use.
- Some procedures can be destructive (volume resets/reformats).
- Always run in isolated certification context.

## References

- Internal hardware certification procedures.
- Exasol deployment and performance best-practice documentation.


