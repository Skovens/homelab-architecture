---
title: Homelab Architecture
---

# Homelab Architecture

I run a homelab. Not just services on a server — an architecture. Every technology choice, every configuration, every deployment pattern is intentional. This site documents what I built, how I built it, and most importantly, why.

## Why This Exists

I transitioned from industrial automation to infrastructure engineering. The homelab is where I practice, experiment, and prove that I can design, deploy, and maintain production-grade systems. This site is both a portfolio and a reference — showing my thinking, not just my configs.

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

Each section follows the same pattern:

1. **What** I'm solving
2. **How** I implemented it
3. **Why** this way, not the obvious alternative

Code samples are concrete but redacted — real config patterns, sanitized paths and domains. The [Decisions](decisions/index.md) section contains architecture decision records explaining the reasoning behind each major choice.
