---
title: Three-Layer Strategy
---

# Three-Layer Backup Strategy

## What

Data is protected by three independent layers: ZFS snapshots, ZFS replication to a DR host, and cloud backup via rclone.

## How

### Layer 1: ZFS Snapshots (Sanoid)

On the home server, sanoid creates hourly/daily/weekly/monthly snapshots of critical datasets. Retention policy:

- 24 hourly snapshots
- 7 daily snapshots
- 4 weekly snapshots
- 6 monthly snapshots

Snapshots are local — they protect against accidental deletion, corruption, or ransomware. They do not protect against hardware failure.

### Layer 2: ZFS Replication (Syncoid)

Every 2 hours, the DR host pulls snapshots from the home server via syncoid over SSH. The replication is pull-based — the DR host initiates, not the home server.

This protects against hardware failure. If the home server dies, the DR host has a recent copy of all critical data.

### Layer 3: Cloud Backup (Rclone)

Every 4 hours, rclone backs up data to pCloud. Two instances (per user) run as separate system users with independent credentials.

This protects against site-wide disaster — fire, theft, flood. If the home and DR hosts are both lost, the cloud backup remains.

## Why

Each layer covers a different failure mode:

| Layer | Protects against | Recovery time | RPO |
|-------|-----------------|---------------|-----|
| Snapshots | Accidental deletion, corruption | Minutes | 1 hour |
| Replication | Hardware failure | Hours | 2 hours |
| Cloud backup | Site-wide disaster | Hours to days | 4 hours |

No single layer is sufficient. Snapshots don't help if the disk is destroyed. Replication doesn't help if both hosts are in the same building. Cloud backup doesn't help if you need to recover a file deleted 30 minutes ago (it's 4 hours stale).

!!! note "RPO vs RTO"
    **RPO (Recovery Point Objective)**: how much data you can afford to lose. My RPO is 2 hours (the replication interval).
    **RTO (Recovery Time Objective)**: how long it takes to restore service. For a file recovery, it's minutes (snapshot rollback). For a full server recovery, it's hours (rebuild from Ansible + restore from DR host).
