---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "Install Patch Builds (for example 7.y.z-p1)"
summary: "Procedure for applying version-specific Exasol hot-fix patch packages and switching database runtime from base version to patch build in EXAoperation."
---

# Install Patch Builds (for example 7.y.z-p1)

## Overview

This guide describes how to install developer-provided patch builds (hot-fixes) for critical issues.

## Versioning Rules

Base package example:

- `EXASolution-7.1.24_x86_64.tar.gz`

Patch package example:

- `EXASolution-7.1.24-p1_x86_64.pkg`

Important:

- Patch builds are version-specific.
- A `7.1.24-p1` patch applies to `7.1.24` and not to other base versions.
- You may need to update to the matching base version first.

## Procedure

1. Stop all databases.
2. Upload patch package in `EXAoperation -> Software` (same workflow as regular updates/security patches).
3. Keep the base version installed (do not remove it).
4. Edit target database configuration and switch runtime version from base to patch build.
5. Start database and validate service health.

## Validation Checklist

- Database runs on expected patched version string.
- Startup/log checks show no package mismatch errors.
- Relevant issue is retested and confirmed resolved.
