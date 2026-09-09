---
title: Cloud Backup
---

# Cloud Backup

## What

Rclone backs up data to pCloud. Each user has their own rclone instance with a separate system user.

## How

Two rclone instances run as separate system users:

| User | Target |
|------|--------|
| `rclone-user-a` | pCloud (user A) |
| `rclone-user-b` | pCloud (user B) |

Each instance:

- Runs as a `nologin` system user (per ADR-0001)
- Has its own lingering systemd session
- Backs up on a 4-hour schedule via systemd user timer
- Is managed via `systemctl --machine=<user>@.host --user`

## Why

Separate rclone instances per user extend the [security model](../../principles/security-model.md) to backup processes: per-service isolation means a compromise of a container running under user A cannot access user B's backup credentials or data. Each user's backup schedule is also independent, so one failure doesn't affect the other.

!!! note "Why pCloud?"
    pCloud offers lifetime plans with client-side encryption. For a homelab backup target, the cost-per-TB is competitive with S3, and there's no egress pricing. The tradeoff is slower restore times compared to a local or VPS-based backup target — which is why ZFS replication to the DR host is the primary backup, and pCloud is the secondary.
