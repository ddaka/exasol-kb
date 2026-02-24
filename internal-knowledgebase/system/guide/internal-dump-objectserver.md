---
tool_name: cos
doc_type: guide
category: system
title: "Internal - Dump objectserver"
summary: "Internal debugging procedure to capture objectserver state via gdb batch script and collect support artifacts."
---
# Internal - Dump objectserver

## Purpose

Capture objectserver internal state for low-level troubleshooting.

## Prerequisites

- Matching DEBUG symbols package for target EXASolution version.
- Root access in COS environment.
- `psh`, `cos_sync_files`, `gdb` available.

## Procedure

1. Transfer and extract debug symbols across nodes:

```shell
cos_sync_files /root/EXASolution-6.0.11-DEBUG-x86_64.tar.gz
psh "tar xf /root/EXASolution-6.0.11-DEBUG-x86_64.tar.gz -C /"
```

2. Create gdb script `/root/script.gdb`:

```text
f 2
p StorageObjects::ObjectTable::print()
quit
```

3. Sync gdb script:

```shell
cos_sync_files /root/script.gdb
```

4. Run batch gdb against objectserver process on nodes:

```shell
psh 'pgrep objectserver | xargs /usr/opt/EXASuite-6/EXARuntime-6.0.11/bin/gdb /usr/opt/EXASuite-6/EXASolution-6.0.11/bin/objectserver -batch -x /root/script.gdb -pid'
```

5. Collect support bundle:

```shell
get_support_info -s <start_date> -e "$(dwad_client shortlist)" -x3
```

## Notes

- Adjust EXASuite/EXASolution paths to match target version.
- This is an internal debug workflow; use only for guided investigations.

## De-duplication note

General coredump/log collection references:

- `documents/cos/confd-system-and-infrastructure.md`
- `documents/cos/cos_directory0structure.md`


