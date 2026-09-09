# Documentation Refactor Plan

## 1. Objective

Transition the documentation from a collection of "standalone essays" to a
structured, hierarchical technical manual. This refactor aims to eliminate
conceptual redundancy, separate architectural "Why" from technical "How," and
provide a clear, welcoming path for new readers.

## 2. New Information Architecture (IA)

### Layer 0: Principles (The "Why")

Consolidates the philosophy and core architectural justifications. These pages
provide the mental model.

- `docs/principles/philosophy.md` — High-level vision/values
- `docs/principles/security-model.md` — Consolidated logic on Rootless, UID
  isolation, and blast radius
- `docs/principles/orchestration-model.md` — Consolidated logic on systemd,
  daemonless, and declarative patterns

### Layer 1: Foundations (The "Bedrock")

The underlying infrastructure that supports all workloads.

- `docs/foundations/networking/` — Mesh VPN, Firewalls, Reverse Proxy
- `docs/foundations/storage/` — ZFS, Snapshots, Replication, Cloud Backup

### Layer 2: Workloads & Implementation (The "How")

Technical manuals for running services. These pages focus on configuration and
assume the "Why" is understood via links to Layer 0 or Layer 4.

- `docs/workloads/containerization/` — Podman, Quadlets, Per-Service Users
- `docs/workloads/ai-stack/` — GPU, Inference, Model Proxy, etc.
- `docs/workloads/services/` — Forgejo, NFS, etc.

### Layer 3: Operations & Automation (The "Management")

How the environment is maintained and recovered.

- `docs/operations/iac/` — Ansible, Secrets, Linting
- `docs/operations/backup-dr/` — Three-Layer Strategy, DR Host

### Layer 4: Architecture Decision Records (The "Deep Dives")

The historical record of specific technical trade-offs.

- `docs/decisions/` — All existing and new ADRs

## 3. Content Migration & Cleanup Strategy

| Current File | Action | New Destination |
| :--- | :--- | :--- |
| `docs/index.md` | **Keep** (Home) | `docs/index.md` |
| `docs/philosophy.md` | **Split**: Keep vision; move security/orchestration logic to Principles. | `docs/principles/` |
| `docs/containerization/index.md` | **Move & Refactor** | `docs/workloads/containerization/index.md` |
| `docs/containerization/podman-over-docker.md` | **Refactor**: Remove "Why Docker is bad"; focus on Podman config; link to ADR. | `docs/workloads/containerization/` |
| `docs/containerization/quadlets.md` | **Refactor**: Remove "Why not Compose"; focus on syntax; link to ADR. | `docs/workloads/containerization/` |
| `docs/containerization/per-service-users.md` | **Refactor**: Remove argument for UID isolation; focus on Ansible implementation; link to ADR-0001. | `docs/workloads/containerization/` |
| `docs/containerization/the-ai-services-pod.md` | **Refactor**: Move "Why" logic to ADR; focus on service discovery/config. | `docs/workloads/ai-stack/` |
| `docs/containerization/egress-gate.md` | **Move**: Reference ADR-0011. | `docs/workloads/containerization/` |
| `docs/ai-stack/*` | **Move** (preserve relative structure) | `docs/workloads/ai-stack/` |
| `docs/services/*` | **Move** | `docs/workloads/services/` |
| `docs/networking/*` | **Move** | `docs/foundations/networking/` |
| `docs/storage/*` | **Move** | `docs/foundations/storage/` |
| `docs/infrastructure-as-code/*` | **Move** | `docs/operations/iac/` |
| `docs/backup-disaster-recovery/*` | **Move** | `docs/operations/backup-dr/` |
| `docs/decisions/*` | **Keep** | `docs/decisions/` |

## 4. Linking Convention

To maintain a welcoming and non-redundant atmosphere, adopt the following
linking standard:

- **Implementation -> Principle:** "Following our
  [Security Model](docs/principles/security-model.md), we configure..."
- **Implementation -> Decision:** "This implementation follows
  [ADR-0001: Rootless Users](docs/decisions/0001-rootless-per-service-users.md)."

## 5. Execution Phases

1. **Phase 1: The Skeleton.** Reorganize the directory structure and update
   `zensical.toml` (navigation).
2. **Phase 2: The Principles.** Create the new `docs/principles/` files by
   consolidating existing "Why" content.
3. **Phase 3: The Technical Manuals.** Iterate through `workloads/` and
   `foundations/`, stripping out redundant arguments and adding the new linking
   convention.
4. **Phase 4: The Decision Audit.** Ensure all unique technical arguments are
   captured in `docs/decisions/`.
5. **Phase 5: Final Polish.** Verify all links, check for tone consistency, and
   run a final walkthrough.

## 6. Status

- [x] Phase 1: Skeleton
- [x] Phase 2: Principles
- [x] Phase 3: Technical Manuals
- [x] Phase 4: Decision Audit
- [x] Phase 5: Final Polish

## 7. Refactor Notes

- **New directory structure** is layered (Principles, Foundations, Workloads,
  Operations, Decisions) matching section 2.
- **Two foundational ADRs were created** that had no previous home:
  - ADR-0012: Podman for rootless containers
  - ADR-0013: Quadlets over Docker Compose
- **Homepage** updated to describe the four-layer reading model and link to the
  Principles section.
