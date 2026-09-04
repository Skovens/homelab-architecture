---
title: Toggleable Services
---

# Toggleable Services

## What

Jellyfin (media server) and Home Assistant (smart home) are deployed but disabled by default. A single flag enables them.

## How

Each service has an `enabled` flag in its role defaults:

```yaml
# roles/jellyfin-container/defaults/main.yml
jellyfin_enabled: false

# roles/homeassistant/defaults/main.yml
homeassistant_enabled: false
```

To enable, override in `group_vars`:

```yaml
jellyfin_enabled: true
homeassistant_enabled: true
```

The playbook conditionally runs the role based on this flag.

## Why

Not every service needs to run all the time. Jellyfin and Home Assistant are services I use occasionally — they don't need to consume resources 24/7.

The toggle pattern means the roles are always present in the repo (tested, maintained, ready), but the services only run when explicitly enabled. This is different from removing the role entirely — the code stays current, and enabling the service is a one-line change.

!!! note "Home Assistant USB passthrough"
    Home Assistant requires USB device passthrough (`/dev/ttyUSB0`) for Zigbee communication. This is configured in the quadlet only when `homeassistant_enabled: true` — the device passthrough directive is conditional.
