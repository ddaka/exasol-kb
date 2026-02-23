---
tool_name: confd_client
doc_type: guide
category: gpu
technical_entities:
  - GPU
  - NVIDIA
  - driver
summary: >
  How to install NVIDIA GPU drivers and Container Toolkit on Exasol cluster
  hosts for GPU-accelerated UDFs.
---

# Install GPU Drivers and Software

## Prerequisites

- Host systems meet GPU system requirements
- Each host updated with latest OS kernel and packages, then restarted
- Root or sudo privileges

## Installation

Install the NVIDIA GPU driver and Container Toolkit on each host using the
Linux distribution package manager:

- **Ubuntu**: See NVIDIA documentation for Ubuntu driver installation
- **RHEL**: See NVIDIA documentation for RHEL driver installation
