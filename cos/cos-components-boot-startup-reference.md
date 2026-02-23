---
title: COS Components, Database Startup, and Boot Sequence
description: Reference summary of key COS services, Exasol DB process startup order, and node boot stages.
tool_name: cos
doc_type: concept
category: COS Architecture
subcommands:
  - cosps
  - csinfo
  - csrec
  - cosexec
---

# COS Components, Database Startup, and Boot Sequence

## 1) Core COS Components

- `cored` (`cos_cored`): core cluster daemon, PID 1 in COS namespace, logs in `/var/log/cored/`
- `logd`: centralized cluster log aggregation, queried via `logd_client`/`logd_collect`
- `lockd`: global lock/barrier coordination across nodes
- `dwad`: database lifecycle and automatic recovery management
- `appserverd` / EXAoperation: management UI and PXE/boot orchestration
- `cos-storage` / SDFS: storage backend for DB files and backups

## 2) Typical Database Process Tree

Common process progression:

`controller` -> `pddserver` -> `objectserver` -> `exasqllog` -> `loaderd` -> `connectionserver` -> `exasql`

Actual runtime trees vary by workload and enabled features, but this sequence is a useful baseline for incident triage.

## 3) Simplified Database Startup Sequence

1. DWAd starts `controller`
2. Controller starts `pddserver`
3. `pddserver` reads metadata from storage
4. Controller starts remaining core services:
   - `objectserver`
   - `logserver`
   - `loaderd`
   - `connectionserver`
   - ETL components
   - SQL worker processes

## 4) Boot Process Stages

### Stage 1: Network/PXE Bootstrap

- Node obtains IP and boot artifacts (kernel + ramdisk) from management node (DHCP/PXE).

### Stage 2: Host Preparation

- Validate node identity/MAC.
- Apply boot flags.
- Initialize disks/partitions.
- Install required OS packages.

### Stage 3: COS Runtime Join

- Start `cored`.
- Join cluster partitions.
- Start `lockd`, `dwad`, `logd`, and plugins.
- Finalize node readiness.

## 5) Operational Use

For startup/boot incidents, pair this sequence with:

- process checks (`cosps -r`, `cosps -N`)
- storage state (`csinfo -v`, `csrec -l`)
- cluster-wide logs (`logd_collect <Service>`)
- node diagnostics (`dmesg`, boot logs)
