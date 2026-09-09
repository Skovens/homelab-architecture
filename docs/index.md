---
title: Homelab Architecture
---

# Homelab Architecture

This site documents my homelab—an evolving environment where I explore infrastructure design and system patterns. Rather than just running services, I use this space to experiment with intentional architecture, documenting not just what I build, but the reasoning behind every configuration and deployment.

## Why This Exists

My journey from industrial automation to infrastructure engineering has made me a lifelong student of how systems work. I view this homelab as a personal laboratory: a safe place to practice production-grade patterns, test new technologies, and learn from both successes and failures.

I'm sharing this site as a living journal of my technical journey. It is part personal reference and part resource for anyone else interested in the "why" behind the "how."

## Hardware

### Home Server

| Component | Spec |
|-----------|------|
| CPU | AMD Ryzen 5 3600 |
| Board | ASRockRack X570D4U |
| RAM | 128 GB DDR4-3200 |
| GPU | NVIDIA Tesla P40 (24 GB VRAM) |
| OS | Debian 13 |
| Storage | ZFS (`bulkdata/` pool) |

### Disaster Recovery Host

| Component | Spec |
|-----------|------|
| Board | RockPro64 (ARM64, RK3399) |
| OS | FreeBSD 15.0 (ZFS root) |
| Storage | `zpool/backup` |

### Reverse Proxy

| Component | Spec |
|-----------|------|
| OS | Debian 13 (VPS) |

## How to Read This Site

The site is organised into four layers:

1. **Principles** — the [philosophy](principles/philosophy.md) and the reasoning behind the security and orchestration choices.
2. **Foundations & Workloads** — how the pieces are built and run: networking, storage, containerization, AI services, and the other services.
3. **Operations** — how the environment is maintained, automated, and recovered.
4. **Decisions** — deep dives into specific technical trade-offs in the [Architecture Decision Records](decisions/index.md).

Code samples are real-world patterns, though paths and sensitive domains are sanitized. Where a page makes a choice, it links to the principle or decision record that explains why — so the reasoning is never repeated and never lost.

