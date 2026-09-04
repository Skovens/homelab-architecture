---
title: "ADR-0008: NVIDIA R580 Driver"
---

# ADR-0008: NVIDIA R580 via `.run` installer + APT pinning

## Status

Accepted

## Context

The GPU is a **Tesla P40 (Pascal, GP102)**. Three constraints collide:

| Constraint | Impact |
|------------|--------|
| NVIDIA dropped Pascal support from R590+ | Cannot use R590 or newer |
| Containers use `server-cuda12` images built against CUDA 12.8 | Requires driver ≥ R570 |
| Debian 13 `non-free` ships only R550 (CUDA 12.4) | Too old for CUDA 12.8 containers |

The intended solution was NVIDIA's official `debian13` CUDA repo pinned to R580. **It failed in practice:** NVIDIA's `debian13` repo does not contain R580 packages (only R590+). The only way to obtain R580 on Debian 13 is NVIDIA's `.run` installer (`NVIDIA-Linux-x86_64-580.173.02.run`).

## Decision

Install the R580 driver from the NVIDIA `.run` installer, not from APT:

- The role downloads and runs `NVIDIA-Linux-x86_64-580.173.02.run` with DKMS (kernel headers metapackage for future upgrades).
- The old cobbled CUDA repo (Debian 12 variant, SHA1 key) is purged; Debian 13's Sequoia-PGP rejects SHA1 since 2026-02-01.
- `nvidia-container-toolkit` from its official repo; CDI generated via `nvidia-ctk config` (`mode=cdi`, `no-cgroups`) for rootless Podman GPU passthrough.
- APT preferences pin any remaining `nvidia`/`cuda` packages to version `580*` (Pin-Priority 1001, downgrade allowed) so tooling never pulls an unsupported branch.
- A manual cleanup + reboot phase runs on the host before the playbook (a loaded driver cannot be purged mid-session).

## Consequences

- **Positive:** P40 gets CUDA 12.8-compatible R580 acceleration; CDI exposes `nvidia.com/gpu=all` to rootless containers; llama.cpp runs on GPU.
- **Negative:** The `.run` installer is not apt-managed (updates are manual); requires an out-of-band host prep step; upgrading the GPU later requires revisiting this ADR.

## Related

- ADR-0001 (rootless container GPU passthrough)
