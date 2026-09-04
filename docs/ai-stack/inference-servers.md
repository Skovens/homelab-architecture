---
title: Inference Servers
---

# Inference Servers

## What

llama.cpp runs twice — once on GPU for fast inference, once on CPU for a dense 256K-context model that doesn't fit in VRAM. Both run as rootless containers in the AI services pod.

## How

### GPU Instance (port 8090)

Runs `ghcr.io/ggml-org/llama.cpp:server-cuda12` with CUDA acceleration. Models:

| Model | Quant | ctx-size | Notes |
|-------|-------|----------|-------|
| Gemma 4 MoE | UD-Q6_K_XL | 262144 | cpu-moe, MTP external drafter |
| Qwen 3.6 MoE | UD-Q6_K_XL | 262144 | cpu-moe, f16 KV, internal MTP |
| Qwen 3.8 Dense | UD-Q3_K_XL | 262144 | q8_0 KV, internal MTP, ubatch=768 |

### CPU Instance (port 8091)

Runs `ghcr.io/ggml-org/llama.cpp:server` (CPU-only, no CUDA). Serves the same Qwen 3.8 Dense model at Q3_K_XL quantization — text-only, no multimodal.

### Key Techniques

**MTP (Multi-Token Prediction) Speculative Decoding** — roughly doubles generation speed. The model predicts multiple tokens per forward pass, then verifies them in parallel. On the P40, this takes TG from ~15 t/s to ~31 t/s for Gemma 4.

**cpu-moe (Mixture of Experts on CPU)** — routes MoE expert FFN layers to DDR4 while keeping attention on GPU. This allows larger MoE models to fit in 24 GB VRAM by offloading the sparse expert layers to system RAM.

### Model Downloads

Models are downloaded via the `hf` CLI (Hugging Face) installed under the `ai-services` user via `pipx`. Downloads are idempotent — `args.creates` prevents re-downloading existing files.

## Why

Two instances because no single configuration handles both workloads:

- **GPU instance**: fast inference for interactive use (chat, code generation). Needs CUDA, needs VRAM.
- **CPU instance**: slow but capable inference for the dense 256K-context model. This model is too large for the P40's 24 GB VRAM at usable precision. On CPU it runs at ~2-4 t/s — slow, but functional for long-context tasks where latency doesn't matter.

The cpu-moe technique is the key innovation: instead of running the entire MoE model on GPU (which would require 48+ GB VRAM), only the attention layers stay on GPU. The expert FFN layers — which are sparse and don't all activate for every token — run on CPU. This fits a 35B-parameter MoE model in 24 GB VRAM.
