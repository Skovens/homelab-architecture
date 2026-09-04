---
title: "ADR-0002: User-scoped systemd management"
---

# ADR-0002: User-scoped systemd via `systemctl --machine=`

## Status

Accepted

## Context

Per-service `nologin` users (ADR-0001) mean container quadlets are user-scoped systemd units in `~/.config/containers/systemd/`. Ansible needs to `daemon-reload`, start, restart, and stop these units. Every "native" approach fails:

- `ansible.builtin.systemd` with `scope: user` — root cannot connect to another user's D-Bus socket regardless of `XDG_RUNTIME_DIR`. This is a kernel-level socket permission block (`Operation not permitted`).
- `sudoers env_keep` — same kernel-level restriction, not a sudo configuration issue.
- `become_method: machinectl` — requires a valid login shell; `nologin` users fail.
- `sudo -u <nologin_user> systemctl --user` — sudo does not re-run PAM, so the D-Bus session bus address is never set.

The `systemctl --machine=<user>@.host --user` pattern works because it routes through the system bus (PID 1) via D-Bus, which root has access to, bypassing the per-user socket entirely. The `community.podman` collection has no quadlet/systemd integration.

## Decision

Manage all user-scoped quadlet units through `systemctl --machine={{ user }}@.host --user` in Ansible `command` tasks and handlers. The pattern is centralized in `roles/podman-container-base/`:

- The role owns user creation, lingering, and per-user systemd directory setup.
- Every container role deploys its `.container` quadlet and runs an explicit `daemon-reload` + `start` on first deploy (bootstrap), with restart handlers wired through `--machine=`.
- **Role-first, not module.** A custom `podman_quadlet`/`systemd_user_unit` Ansible module is deferred until the true API is proven by repeated real-world usage across the role set. If a generic user-scoped-systemd pattern emerges, promote it to a module then.

## Consequences

- **Positive:** One consistent, working pattern for all rootless quadlet roles; no new dependencies.
- **Negative:** Uses raw `command`/`shell` — no native `systemd` module semantics (`state`, `enabled`, change detection); first-deploy bootstrap requires explicit tasks, not just handlers.
- **Revisit:** If operational cost outweighs the security benefit of per-service users (ADR-0001), re-evaluate rootful Podman with system-level quadlets, which restores native `systemd` module use.

## Related

- ADR-0001, ADR-0003
