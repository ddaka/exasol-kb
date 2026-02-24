---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "Prepare a PoC Cluster Before First Query Execution"
summary: "Operational checklist for technical and customer-facing readiness steps before starting first-query activity in an Exasol PoC environment."
---

# Prepare a PoC Cluster Before First Query Execution

## Overview

This checklist helps ensure a PoC cluster is technically ready and customer onboarding communication is complete before first workload execution.

## Technical Readiness Checklist

- Auditing enabled.
- Profiling enabled.
- Query cache strategy decided (`on` or `off` per PoC goals).
- Backup schedule configured.
- First backup completed successfully.
- Monitoring services configured and validated.
- Users, schemas, and privileges prepared.
- Source DDL converted for Exasol compatibility.
- Data loading path validated.
- EXAoperation access verified.

Related guide:

- [Default configuration of a remote PoC and access to test data](default-configuration-of-a-remote-poc-and-access-to-test-data.md)

## Customer Communication Checklist

- Share login credentials securely.
- Share cluster architecture overview.
- Share hardware overview.
- Provide recommended client tooling guidance (for example MobaXterm on Windows).
- Provide EXAplus favorites/templates for standard diagnostics (profiles, sessions, maintenance queries).

## Exit Criteria

PoC can proceed when technical checks pass and customer has all required access artifacts and onboarding material.
