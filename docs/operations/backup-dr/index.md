---
title: Backup & Disaster Recovery
---

# Backup & Disaster Recovery

Data integrity is non-negotiable. This section outlines the multi-layered strategy used to protect the homelab against everything from accidental `rm -rf` to total site-wide failure.

## The Strategy

We follow a tiered approach to redundancy, moving from high-frequency, low-latency protection to low-frequency, high-latency catastrophic recovery.

```mermaid
graph TD
    D[pCloud] -->|Rclone Sync \(one-way\)| B[Homeserver - ZFS]
    B -->|Syncoid Replication \(one-way\)| C[DR Host - Rockpro]
```

## Core Components

- [Three-Layer Strategy](three-layer-strategy.md) — The philosophy of tiered redundancy.
- [DR Host](dr-host.md) — Details on the FreeBSD-based recovery target and its role in the replication loop.
