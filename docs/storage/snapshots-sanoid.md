---
title: Snapshots with Sanoid
---

# Snapshots with Sanoid

## What

Sanoid provides policy-driven ZFS snapshot management. I use it to automatically create and prune snapshots on a schedule.

## How

Sanoid is deployed on both the home server and the DR host. Configuration is template-driven:

```ini title="sanoid.conf"
[bulkdata/backups]
use_template = production
recursive = yes

[template_production]
frequently = 0
hourly = 24
daily = 7
weekly = 4
monthly = 6
yearly = 0
autosnap = yes
autoprune = yes
```

On FreeBSD (DR host), sanoid runs via cron every minute:

```cron
* * * * * sanoid --cron
```

On Debian (home server), a systemd timer handles scheduling.

Sanoid is applied to:

- `bulkdata/backups` — replicated backup data
- `bulkdata/podman/forgejo` — self-hosted git data

## Why

Manual snapshot management doesn't scale. Without sanoid, snapshots accumulate until someone remembers to prune them — or they never get pruned at all.

Sanoid's template system means I define retention once and apply it to every dataset. The same template produces hourly, daily, weekly, and monthly snapshots with automatic pruning. When I add a new dataset that needs snapshots, I apply the template and it inherits the entire policy.

!!! note "Why not zfs-auto-snapshot?"
    `zfs-auto-snapshot` is a simpler tool that creates snapshots on a fixed schedule. Sanoid adds template-based retention policies, which means I can say "keep 24 hourly, 7 daily, 4 weekly" instead of "keep the last N snapshots." The policy is explicit, not count-based.
