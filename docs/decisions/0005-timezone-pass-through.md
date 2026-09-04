---
title: "ADR-0005: Timezone Pass-Through"
---

# ADR-0005: Container timezone pass-through

## Status

Accepted

## Context

Hosts run `Europe/Your-Timezone` while containers defaulted to UTC, producing wrong timestamps in logs, UIs, and DB writes. Two mechanisms exist to pass the host timezone into a container:

- Mounting `/etc/localtime` — read by **glibc/Go** images (open-webui, oikb, docling-serve, llamacpp, openhands, jellyfin, github-mcp, forgejo-mcp).
- `Environment=TZ=...` — required by **Alpine/musl** images (searxng, searxng-valkey, forgejo, homeassistant, healthchecks), which ignore `/etc/localtime`.

One open question was whether auth services should be excluded. Research concluded: JWT `iat`/`exp`/`nbf` are **epoch-second based and timezone-invariant** — "token issued in the future" errors come from NTP clock drift, never from a timezone mount. So OAuth/SAML **clients** (open-webui) are safe to include; only **token issuers** are excluded.

## Decision

Every non-auth container quadlet gets **both**:

```ini
Volume=/etc/localtime:/etc/localtime:ro
Environment=TZ=Europe/Your-Timezone
```

The timezone is defined once in `playbooks/group_vars/home_server.yml`. The repo skeleton enforces this via a `rolename_tz_mount` switch (default `true`) in the container skeleton template — set `false` only for auth **token issuers**.

## Consequences

- **Positive:** Correct timestamps in both glibc and musl images with one variable; enforced by the skeleton so new roles get it for free.
- **Negative:** One more mount + env per quadlet; the exclusion rule (token issuers only) must be respected — applying it to a token issuer would risk clock-skew confusion in OIDC debugging.
