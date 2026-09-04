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

Separate rclone instances per user ensure that:

- Backup credentials for user A are not accessible to user B's containers
- Each user's backup schedule is independent
- If one user's backup fails, the other continues unaffected

The same pattern applies as with containers: per-service isolation extends to backup processes. A compromise of a container running under user A cannot access user B's backup credentials or data.

!!! note "Why pCloud?"
    pCloud offers lifetime plans with client-side encryption. For a homelab backup target, the cost-per-TB is competitive with S3, and there's no egress pricing. The tradeoff is slower restore times compared to a local or VPS-based backup target — which is why ZFS replication to the DR host is the primary backup, and pCloud is the secondary.
