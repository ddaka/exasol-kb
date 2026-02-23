---
tool_name: confd_client
doc_type: reference
category: gpu
technical_entities:
  - GPU
  - NVIDIA
  - VRAM
  - LLM
  - machine learning
summary: >
  Hardware sizing guidance for GPU-enabled Exasol nodes — RAM, CPU, and GPU
  sizing, LLM VRAM requirements table, and training vs inference considerations.
---

# GPU Hardware Sizing

## RAM Sizing

Add at least **64–128 GiB RAM** per data node beyond normal BI query
requirements to accommodate UDF workloads.

## CPU Sizing

Plan for **4–8 CPU cores per GPU** in data nodes. Add more cores if running
additional workloads.

## GPU Sizing

Only NVIDIA Data Center GPUs are supported. Different models vary in cores,
VRAM, memory bandwidth, and interconnects (PCIe, NVLink).

### LLM VRAM Requirements

| Model Size              | VRAM Required   | Notes                                     |
|-------------------------|-----------------|-------------------------------------------|
| Tiny (100M–2B)          | 2–4 GiB         |                                           |
| Small (2B–10B)          | 6–16 GiB        |                                           |
| Medium (10B–20B)        | 16–24 GiB       |                                           |
| Large (20B–70B)         | 24–48 GiB       | Single or multiple high-end GPUs          |
| Very Large (70B–110B)   | ≥80 GiB         | Single or multiple high-end GPUs          |
| Super Large (>110B)     | ≥160 GiB        | Multiple GPUs with ≥80 GiB VRAM each     |

Actual VRAM needs are slightly higher due to intermediate results, optimizer
state, and input data buffers.

### Classic Machine Learning

Random forest, linear regression, K-means, PCA — typically the same requirements
as Tiny LLM models. Clustering/dimension-reduction methods may need the whole
dataset in VRAM.

## Training vs Inference

| Stage                | GPU Requirements                                  |
|----------------------|---------------------------------------------------|
| Training/Fine-tuning | High-end GPUs, ample VRAM, high-speed networking  |
| Inference            | Smaller GPUs (T4, A10) often sufficient           |
