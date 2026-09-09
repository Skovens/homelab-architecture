---
title: NFS
---

# NFS

## What

NFS exports media storage from the home server to VPN clients. Restricted to the mesh subnet.

## How

The NFS server role exports `/bulkdata/Media` with the following options:

```yaml
nfs_exports:
  - path: /bulkdata/Media
    clients: "10.0.0.0/24"
    options: "all_squash,anonuid=0,anongid=451"
```

`all_squash` maps all client requests to the anonymous UID/GID. The anonymous GID (451) grants read access to the media dataset.

## Why

NFS is the simplest network file sharing option. The mesh VPN restricts access to the overlay subnet, so no additional access controls are needed on the export itself.

The `all_squash` mapping is critical — without it, each client's local UID would need to match the server's UID for the media dataset. With `all_squash`, every client maps to the same anonymous user regardless of local UID.
