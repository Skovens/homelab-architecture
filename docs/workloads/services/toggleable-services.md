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

Occasional services don't need to consume resources 24/7. The toggle pattern keeps roles present and maintained in the repo while services only run when explicitly enabled — see the [Orchestration Model](../../principles/orchestration-model.md) for the broader deployment philosophy.

!!! note "Home Assistant USB passthrough"
    Home Assistant requires USB device passthrough (`/dev/ttyUSB0`) for Zigbee communication. This is configured in the quadlet only when `homeassistant_enabled: true` — the device passthrough directive is conditional.
