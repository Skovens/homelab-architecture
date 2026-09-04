---
title: DR Host
---

# DR Host

## What

The disaster recovery host runs FreeBSD on a RockPro64 ARM board. It receives ZFS replication from the home server and serves as the recovery target.

## How

| Component | Spec |
|-----------|------|
| Board | RockPro64 (ARM64, RK3399) |
| OS | FreeBSD 15.0 |
| Storage | ZFS root (`zpool`) |
| Firewall | PF |
| VPN | DNClient (same mesh as home server) |
| Fan control | `fand` daemon for RockPro64 |

The DR host runs its own Ansible playbook (`rockpro64.yml`) with roles for:

- ZFS import and configuration
- PF firewall
- DNClient mesh VPN
- Sanoid (for pruning replicated snapshots)
- Syncoid (for receiving replicated snapshots)

## Why

FreeBSD on ARM was chosen for three reasons:

1. **ZFS maturity** — FreeBSD's ZFS implementation is the reference. Linux ZFS (OpenZFS) is a port; FreeBSD ZFS is native.
2. **Low power** — the RK3399 draws ~5W idle. The DR host runs 24/7, so power consumption matters.
3. **Hardware availability** — the RockPro64 was already in my possession from previous projects.

!!! warning "Performance ceiling"
    The RK3399's network stack limits Nebula tunnel throughput to ~89 Mbps. This is a hardware limit — no software optimization can improve it. For a DR host that receives incremental ZFS snapshots, this is sufficient. For a primary server, it would not be.
