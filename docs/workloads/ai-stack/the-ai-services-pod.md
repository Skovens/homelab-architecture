---
title: The AI Services Pod
---

# The AI Services Pod

## What

The AI service family is a deliberate exception to per-service-user isolation. All AI containers share a single user and a single Podman pod.

## How

One user (`ai-services`), one pod (`ai-services.pod`), all AI containers joined to it:

```ini title="ai-services.pod"
[Pod]
PublishPort=

[Install]
WantedBy=default.target
```

Containers join the pod and communicate over localhost:

```ini title="llamacpp.container"
[Container]
Pod=ai-services.pod
Image=ghcr.io/ggml-org/llama.cpp:server-cuda12
PublishPort=8090:8080
```

Service discovery uses localhost with different ports:

| Service | Internal Address |
|---------|-----------------|
| Open WebUI | `localhost:3000` |
| llama.cpp (GPU) | `localhost:8090` |
| llama.cpp (CPU) | `localhost:8091` |
| SearXNG | `localhost:8888` |
| Docling-serve | `localhost:5001` |
| GitHub MCP | `localhost:8082` |
| Forgejo MCP | `localhost:8085` |

Containers declare systemd dependencies to avoid race conditions:

```ini
[Service]
After=ai-services.pod llamacpp.service
Wants=ai-services.pod llamacpp.service
```

## Why

This is a deliberate exception to per-service-user isolation — see [Security Model](../../principles/security-model.md) and [ADR-0003](../../decisions/0003-ai-services-pod.md) for the full reasoning.

!!! note "The exception proves the rule"
    Container-level hardening (`DropCapability=ALL`, `NoNewPrivileges=true`, read-only rootfs, per-container `MemoryMax`) remains in place. The host UID is shared, but the primary isolation layer is user namespaces, not host UIDs.
