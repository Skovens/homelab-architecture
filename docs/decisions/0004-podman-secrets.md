---
title: "ADR-0004: Podman Secrets"
---

# ADR-0004: Podman secrets for service credentials

## Status

Accepted

## Context

Services require credentials: OIDC client secrets, healthchecks `secret_key`, Authelia admin/session/storage/JWT secrets, API tokens (GitHub, Forgejo). Historically these were generated with `lookup()` and written to `.env` files on disk, or inlined into templates. That left plaintext credentials at rest outside any secret store, with no rotation story and a risk of accidental commit.

## Decision

Store all service credentials as **Podman secrets** and inject them via quadlet `Secret=` directives or at container start:

- Secret generation is idempotent: `podman secret inspect <name>` → create on miss with `openssl rand`.
- Secrets are generated at deploy time on the target host, never committed to the repo.
- Secret-material tasks run with `no_log: true`; where a `.env` file is still required, mode `0600`.
- Secrets are scoped to the service user's Podman store (ADR-0001) — per-user isolation preserved.
- Sensitive values that need human-readable storage (e.g., generated admin passwords) are echoed into the secret with stdout suppressed.

## Consequences

- **Positive:** No plaintext secrets on disk or in git; per-service isolation; rotation = recreate the secret + restart.
- **Negative:** Every secret adds a deploy-time generation task; `podman secret` lifecycle must be understood (secrets are deleted with the container unless the store is shared); not all images support secret injection — some still need env files.
- **Migration debt:** remaining `lookup()`/`.env` usages in roles should be audited and moved to Podman secrets.

## Related

- ADR-0001 (per-user secret stores)
