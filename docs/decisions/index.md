---
title: Architecture Decision Records
---

# Architecture Decision Records

ADRs document the reasoning behind major architectural choices. Each record captures the context, the decision, and the consequences — so the "why" is never lost.

| ADR | Decision | Status |
|-----|----------|--------|
| [ADR-0001](0001-rootless-per-service-users.md) | Rootless per-service `nologin` users for Podman | Accepted |
| [ADR-0002](0002-user-scoped-systemd-machine.md) | User-scoped systemd via `systemctl --machine=` | Accepted |
| [ADR-0003](0003-ai-services-pod.md) | Shared `ai-services` pod for AI containers | Accepted |
| [ADR-0004](0004-podman-secrets.md) | Podman secrets for service credentials | Accepted |
| [ADR-0005](0005-timezone-pass-through.md) | Container timezone pass-through | Accepted |
| [ADR-0006](0006-oikb-docling-filter.md) | oikb sources limited to docling-supported types | Accepted |
| [ADR-0007](0007-docling-sync-timeout.md) | Docling sync timeout raised instead of PDF size cap | Accepted |
| [ADR-0008](0008-nvidia-r580-run-installer.md) | NVIDIA R580 via `.run` installer + APT pinning | Accepted |
| [ADR-0010](0010-fixed-context-sizes-replace-fit.md) | Fixed context sizes replace `--fit` auto-fitting | Accepted |
| [ADR-0011](0011-egress-gate-for-rootless-containers.md) | Egress gate for rootless containers | Accepted |
