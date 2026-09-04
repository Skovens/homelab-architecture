---
title: Replication with Syncoid
---

# Replication with Syncoid

## What

Syncoid handles ZFS replication over SSH. I use it for disaster recovery — the DR host pulls snapshots from the home server every 2 hours.

## How

The replication is **pull-based**: the DR host (FreeBSD on RockPro64) initiates the pull, not the home server pushing.

On the DR host, cron runs every 2 hours:

```cron
0 */2 * * * /usr/local/bin/syncoid --no-privilege-elevation --no-sync-snap --no-rollback -r zfs-send@100.100.100.4:bulkdata/backups zpool/backup
```

ZFS delegation enables non-root replication. Each side has a dedicated user with specific permissions:

| Host | User | Permissions | Dataset |
|------|------|-------------|---------|
| Home server | `zfs-send` | `snapshot,send,hold,destroy` | `bulkdata/backups` |
| DR host | `zfs-recv` | `create,receive,mount,rollback,destroy` | `zpool/backup` |

## Why

Pull-based replication has a key advantage: the DR host controls when replication happens. If the home server is under load, the DR host can back off. If the home server is offline, the DR host retries on schedule.

Push-based replication would require the home server to maintain a list of DR targets and manage SSH keys for each one. Pull-based means the DR host is self-contained — it knows where to pull from, and the home server doesn't need to know the DR host exists.

!!! warning "Performance ceiling"
    The Nebula tunnel between hosts maxes out at ~89 Mbps on the RockPro64's RK3399. This is a hardware limit — the RK3399's network stack cannot push more data through the tunnel. No software optimization can improve this.
