---
title: Secrets Management
---

# Secrets Management

## What

Service credentials are stored as Podman secrets, not `.env` files. Secrets are generated at deploy time, never committed to the repo.

## How

Secret creation is idempotent:

```yaml
- name: Check if secret exists
  command: podman secret inspect myservice-secret
  register: secret_check
  failed_when: false
  changed_when: false

- name: Create secret
  command: podman secret create myservice-secret -
  stdin: "{{ lookup('ansible.builtin.password', length=32) }}"
  when: secret_check.rc != 0
  no_log: true
```

Secrets are injected via quadlet `Secret=` directives:

```ini
[Container]
Secret=myservice-secret
```

Or via `EnvironmentFile` for images that don't support secret injection:

```ini
[Container]
EnvironmentFile=%h/secrets.env
```

## Why

`.env` files leave plaintext credentials on disk. If the host is compromised, every `.env` file is readable. Podman secrets are stored in the user's Podman secret store — encrypted at rest, scoped to the user.

The `no_log: true` directive prevents Ansible from logging secret values. Without it, the generated secret would appear in playbook output.

!!! warning "Migration debt"
    Not all roles have been migrated to Podman secrets. Some still use `.env` files with `lookup()` generation. These should be migrated — the pattern is proven and the migration is straightforward.
