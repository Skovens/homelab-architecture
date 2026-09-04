---
title: AI Stack
---

# AI Stack

The AI stack runs entirely on-premises. A Tesla P40 GPU, llama.cpp for inference, Open WebUI as the frontend, and a constellation of MCP servers for tool integration.

- [GPU](gpu.md) — Tesla P40, NVIDIA R580 driver
- [Inference Servers](inference-servers.md) — llama.cpp GPU vs CPU, MTP speculative decoding
- [Model Proxy](model-proxy.md) — llama-swap FIFO scheduler
- [Open WebUI](open-webui.md) — frontend wired via IaC
- [Document Processing](document-processing.md) — docling-serve
- [MCP Servers](mcp-servers.md) — GitHub, Forgejo, Vault tool servers
- [Knowledge Base](knowledge-base.md) — oikb, Obsidian vault sync
