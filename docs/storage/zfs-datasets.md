---
title: ZFS Datasets
---

# ZFS Datasets

## What

Every service with persistent data gets its own ZFS dataset. Datasets are tuned to their workload profile — databases get small record sizes, media gets large ones.

## How

Workload profiles are defined in `playbooks/group_vars/all.yml` and applied when creating datasets via `include_role: zfs-dataset`:

| Profile | recordsize | compression | atime | xattr | Use case |
|---------|-----------|-------------|-------|-------|----------|
| `db` | 16K | lz4 | off | sa | Databases, small random I/O |
| `cache` | 128K | lz4 | off | sa | General container data |
| `media` | 1M | lz4 | off | sa | Large sequential files |

Dataset paths follow the pattern: `bulkdata/podman/<service>/`

```yaml
- name: Create ZFS dataset for myservice
  include_role:
    name: zfs-dataset
  vars:
    zfs_dataset_name: "bulkdata/podman/myservice"
    zfs_profile: cache
```

## Why

ZFS datasets provide:

- **Per-service snapshots** — snapshot one service without affecting others
- **Per-service quotas** — prevent one service from consuming all disk
- **Workload-tuned I/O** — databases don't waste I/O on 1M record sizes
- **Send/receive granularity** — replicate individual datasets, not entire pools

Without per-service datasets, a single `bulkdata/podman` dataset means all services share the same recordsize and compression settings. A database container and a media server would get the same I/O tuning — which is optimal for neither.
