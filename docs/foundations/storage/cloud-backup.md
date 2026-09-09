---
title: Cloud Backup
---

# Cloud Backup

Rclone backs up data to pCloud. Each user has their own rclone instance with a separate system user.

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

I chose pCloud not for its technical superiority, but for its **practicality and transparency**. Because my family already uses pCloud, they can access their data seamlessly without ever needing to interact with the underlying infrastructure or know that automated backups are running in the background. This minimizes "technical friction" during a recovery event.

This approach involves a trade-off: because pCloud is a client-side tool, I lose some of the native ZFS efficiencies (like optimized recordsize handling) during read/write operations. However, on the push side, the ability to [rollback ZFS datasets](https://github.com/openzfs/zfs/blob/master/docs/rollback.md) ensures that if a sync captures encrypted or corrupted data, I can quickly revert to a known good state before the cloud copy is overwritten.

While local snapshots and the DR host protect against software errors or single-host failure, the DR host is located at a separate, undisclosed site. Cloud backup provides protection against even more catastrophic site-wide disasters.

