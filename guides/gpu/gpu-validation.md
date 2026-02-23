---
tool_name: confd_client
doc_type: guide
category: gpu
technical_entities:
  - GPU
  - NVIDIA
  - nvidia-smi
  - nvidia-container-cli
  - CUDA
  - EXA_METADATA
  - DKMS
summary: >
  How to validate GPU setup in Exasol — nvidia-smi driver check,
  nvidia-container-cli verification, EXA_METADATA query, and troubleshooting
  DKMS driver issues.
---

# Validate GPU Setup

## Driver Validation

Check driver version, persistence mode status, and GPU list:

```bash
nvidia-smi --query-gpu=name,driver_version,index,persistence_mode --format=csv
```

Expected output:

```
name, driver_version, index, persistence_mode
Tesla T4, 535.247.01, 0, Enabled
```

## Container Tooling Validation

Verify `nvidia-container-cli` recognises installed GPUs:

```bash
nvidia-container-cli info
```

Expected output:

```
NVRM version:   535.247.01
CUDA version:   12.2

Device Index:   0
Device Minor:   0
Model:          Tesla T4
Brand:          Nvidia
GPU UUID:       GPU-43aae450-f01d-5854-12d2-7ae261f27920
Bus Location:   00000000:00:1e.0
Architecture:   7.5
```

## Verify GPU Acceleration in Database

Query `EXA_METADATA` (requires a running database):

```sql
SELECT PARAM_NAME, PARAM_VALUE
FROM EXA_METADATA
WHERE PARAM_NAME = 'acceleratorDeviceDetected';
```

Result `acceleratorDeviceDetected = 1` means GPUs are detected and ready.

## Troubleshooting

### nvidia-smi Fails

**Error:**

```
NVIDIA-SMI has failed because it couldn't communicate with the NVIDIA driver.
Make sure that the latest NVIDIA driver is installed and running.
```

**Cause:** Driver kernel modules not correctly built/installed.

**Solution:**

1. Check DKMS module status:

```bash
uname -r
dkms status
```

2. Verify `nvidia/<FULL-DRIVER-VERSION>` shows as `installed` for current kernel

3. If missing, reinstall:

```bash
sudo dkms autoinstall
```

4. Restart the system after reinstalling DKMS modules

If unresolved, reinstall GPU drivers after updating the OS kernel and packages.
