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

NFS is the simplest way to share files over the network. The mesh VPN means the export is only accessible to hosts on the overlay — no need for complex access controls.

`all_squash` is critical. Without it, the client's UID would need to match the server's UID for the media dataset. With `all_squash`, every client maps to the same anonymous user, regardless of their local UID.
