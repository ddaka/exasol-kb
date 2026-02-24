---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "Internal - Enable PXE for HP 10G cards"
summary: "Enable PXE boot on HP 10G adapters using Intel BOOTUTIL from DOS media."
---
# Internal - Enable PXE for HP 10G cards

## Purpose

Enable PXE boot functionality on HP 10G NICs for provisioning/reinstall workflows.

## Prerequisites

- DOS boot image containing `BOOTUTIL.EXE` (example: `fd11src.iso`).
- Physical/remote console access.

## Procedure

1. Boot node from DOS ISO.
2. Open DOS prompt.
3. Enable NIC flash access:

```text
BOOTUTIL.EXE -ALL -FLASHENABLE
```

4. Reboot.
5. Enable PXE boot on all detected NICs:

```text
BOOTUTIL.EXE -ALL -BOOTENABLE=PXE
```

6. Reboot.
7. Use one-time boot menu (`F11`) and boot once via PXE option.
8. Configure BIOS boot order to include PXE 10G interfaces.

## Important note

Node must be booted once from the `F11` one-time PXE menu; otherwise PXE interfaces may not appear in permanent BIOS boot-order settings.

## Downloads

- <https://github.com/exasol/Internal-Knowledgebase/files/9990382/fd11src_part1.zip>
- <https://github.com/exasol/Internal-Knowledgebase/files/9990400/fd11src_part2.zip>


