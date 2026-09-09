---
title: Open WebUI
---

# Open WebUI

Open WebUI is the frontend for all AI services. It connects to both llama.cpp instances, docling-serve for document processing, SearXNG for search, and multiple MCP servers for tool use.

## How

Open WebUI is wired entirely via environment variables in the quadlet — no manual UI configuration:

```ini
[Container]
Image=ghcr.io/open-webui/open-webui:main
Environment=OPENAI_API_BASE_URLS=http://localhost:8090;http://8091
Environment=ENABLE_PERSISTENT_CONFIG=false
```

`ENABLE_PERSISTENT_CONFIG=false` forces environment variables to take precedence over UI settings. This means the Ansible playbook is the single source of truth for configuration.

Tool servers are configured via `open_webui_tool_servers`:

| Tool | Protocol | Endpoint |
|------|----------|----------|
| GitHub MCP | OpenAPI (via mcpo) | `localhost:8082` |
| Forgejo MCP | Streamable HTTP | `localhost:8085/mcp` |
| Vault MCP | Streamable HTTP | `localhost:8095/mcp` |

## Why

`ENABLE_PERSISTENT_CONFIG=false` ensures environment variables always take precedence over UI settings, so the Ansible repo remains the single source of truth. Without it, UI changes override the repo and break reproducibility. See [Orchestration Model](../../principles/orchestration-model.md).

!!! note "Ordering constraint"
    Open Terminal must run before Open WebUI in the playbook. Open Terminal creates an API key secret that Open WebUI reads at startup. If Open WebUI starts first, the secret doesn't exist yet.

!!! tip "Secrets Management"
    The Ansible play does not contain any secrets. If a required secret is missing, the play will pause and prompt for it, providing precise instructions on how to obtain it. This ensures that even months later, the process for generating and providing secrets remains clear and repeatable.

