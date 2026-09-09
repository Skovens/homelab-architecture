---
title: GPU
---

# GPU

## What

An NVIDIA Tesla P40 provides 24 GB VRAM for LLM inference. It runs the R580 driver via the `.run` installer — the only way to get CUDA 12.8-compatible drivers on Debian 13 for a Pascal GPU.

## How

The Tesla P40 is a datacenter GPU — no display output, passive cooling, 250W TDP. It sits in an ASRockRack X570D4U board with a Ryzen 3600 and 128 GB DDR4-3200.

Driver installation:

- NVIDIA R580 (`NVIDIA-Linux-x86_64-580.173.02.run`) via `.run` installer with DKMS
- `nvidia-container-toolkit` from official repo
- CDI configured for rootless Podman: `nvidia-ctk config --cdi-default --no-cgroups`
- APT pinned to `580*` to prevent accidental upgrades

GPU management tools:

- `nvidia-pstated` — power state management
- `gpu-fancontrol` — IPMI-based fan control (passive card needs external cooling)
- `gpu-night-mode` — scheduled power limit (125W) for quiet nighttime operation

## Why

NVIDIA dropped Pascal support from R590+, and Debian 13 only ships R550 (too old for CUDA 12.8 containers). The `.run` installer is the only path to R580 on Debian 13. See [ADR-0008](../../decisions/0008-nvidia-r580-run-installer.md).

!!! warning "R580 is a dead end"
    R580 is the last driver to support Pascal. If the P40 dies, the replacement GPU must be Ampere or newer (which supports R590+). The entire driver installation process changes for newer GPUs.
