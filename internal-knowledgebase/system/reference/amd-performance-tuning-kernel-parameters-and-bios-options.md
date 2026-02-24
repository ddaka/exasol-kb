---
tool_name: cos
doc_type: reference
category: system
title: "AMD performance tuning: kernel parameters and BIOS options"
summary: "Reference baseline for AMD platform BIOS and kernel tuning used in Exasol performance-oriented deployments."
---
# AMD performance tuning: kernel parameters and BIOS options

## Scope

Reference baseline for AMD-based systems where deterministic high performance is required.

## Kernel boot options

Example `/etc/cos/boot_options` baseline:

```text
iommu=pt cpufreq.default_governor=performance processor.max_cstate=1 intel_idle.max_cstate=0
```

Use with latest validated BIOS/firmware.

## BIOS baseline categories

Typical categories to verify:

- Integrated devices / PCI preferred I/O settings.
- System profile (custom performance profile).
- Processor settings (NUMA/core/topology/power states).
- Memory settings (frequency/interleaving/scrub behavior).

## Validation method

Capture effective BIOS values with vendor tooling (for example `racadm`) and compare against approved baseline profile.

## Notes

- Apply only with platform-specific validation.
- Keep profile snapshots before/after change for rollback.
- Re-verify after firmware updates.


