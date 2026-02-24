---
tool_name: c4
doc_type: guide
category: system
title: "SaaS - Get info about c4 global network config file parameters"
summary: "Use `c4 help` to inspect SaaS global-network configuration parameters before deployment."
---
# SaaS - Get info about c4 global network config file parameters

## Purpose

Inspect required C4 configuration parameters used for SaaS global-network deployment.

## Key parameters

- `CCC_SAAS_AWS_GLOBAL_NETWORK`
- `CCC_AWS_TRANSIT_GATEWAY_NAME`
- `CCC_AWS_GLOBAL_NETWORK_NAME`

## Procedure

Run parameter help:

```bash
bin/c4 help saas CCC_SAAS_AWS_GLOBAL_NETWORK
```

Repeat for any other parameter:

```bash
bin/c4 help saas CCC_AWS_TRANSIT_GATEWAY_NAME
bin/c4 help saas CCC_AWS_GLOBAL_NETWORK_NAME
```

## Notes

- The same pattern works for all SaaS config keys:

```bash
bin/c4 help saas <PARAMETER_NAME>
```

- Validate values before deployment to avoid partial or failed global-network provisioning.

## Canonical references

- `documents/c4/c4_best_practices.md`
