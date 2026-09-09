---
title: Ansible Patterns
---

# Ansible Patterns

## What

The Ansible repo follows consistent patterns across all 41 roles: shared base role, tag-based organization, feature flags, and group_vars for overrides.

## How

### Role Structure

Every container role follows the same pattern:

1. Import `podman-container-base` (user setup, lingering, egress gate, image prune)
2. Create ZFS dataset via `include_role: zfs-dataset`
3. Template quadlet files (`.container`, `.pod`, `.network`, `.volume`)
4. Notify handler for restart

The `podman-container-base` role is the shared lifecycle manager. It handles:

- Service user creation with `nologin` shell
- `loginctl enable-linger` for systemd user sessions
- Egress gate deployment (`wait-for-egress` + systemd drop-in)
- Weekly image pruning (`podman image prune -a -f`)

### Tags

Playbooks use tags for selective execution:

| Tag | Scope |
|-----|-------|
| `bootstrap` | User creation, system setup |
| `system` | Kernel parameters, sysctl |
| `networking` | Firewall, VPN |
| `container` | Quadlet deployment |
| `monitoring` | Healthchecks |
| `backup` | Sanoid, syncoid, rclone |

### Feature Flags

Services are toggled via `<service>.enabled | default(false)`:

```yaml
# group_vars/home_server.yml
jellyfin_enabled: false
homeassistant_enabled: false
```

### Group Vars

Configuration is layered:

| File | Scope |
|------|-------|
| `playbooks/group_vars/all.yml` | All hosts |
| `playbooks/group_vars/home_server.yml` | Home server only |
| `playbooks/group_vars/disaster_recovery.yml` | DR host only |
| `playbooks/group_vars/proxy.yml` | Proxy host only |

Role defaults in `roles/<role>/defaults/main.yml` provide the base values. Group_vars override per host.

The rationale for a unified, declarative approach to service deployment is covered in the [Orchestration Model](../../principles/orchestration-model.md).
