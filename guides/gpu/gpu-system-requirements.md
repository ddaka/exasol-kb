---
tool_name: confd_client
doc_type: reference
category: gpu
technical_entities:
  - GPU
  - NVIDIA
  - CUDA
  - UDF
summary: >
  System requirements for GPU acceleration in Exasol — supported NVIDIA GPUs,
  operating systems, and software requirements.
---

# GPU System Requirements

GPU acceleration for user defined functions (UDFs) is available in on-premises
Exasol 2025.2 and later. A technology preview is available in 2025.1 (contact
Support to enable).

Cluster nodes must meet the general Exasol system requirements, plus:

## Hardware Requirements

- Only **NVIDIA Data Center GPUs** are supported
- The number and model of GPU devices must be **identical on all nodes**
- NVIDIA Multi-Instance GPU (MIG) is **not supported**
- Enable GPU ECC memory mode for security

## Supported Operating Systems

| Distribution                    |
|---------------------------------|
| Ubuntu 24.04 LTS                |
| Ubuntu 22.04 LTS                |
| Ubuntu 20.04 LTS                |
| Red Hat Enterprise Linux 9      |
| Red Hat Enterprise Linux 8      |

## Software Requirements

| Component                  | Requirement                                                |
|----------------------------|------------------------------------------------------------|
| NVIDIA GPU Driver          | LTS branch only; keep updated to latest supported version  |
| NVIDIA Container Toolkit   | Newest version, pinned to installation version             |
