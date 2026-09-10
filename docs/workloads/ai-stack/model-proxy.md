---
title: Model Proxy
---

# Model Proxy

llama-swap sits in front of the llama.cpp instances and manages model switching. It prevents aborted in-flight streaming responses when Open WebUI switches models.

## How

llama-swap runs as a pure HTTP proxy (no GPU needed) on port 8092. It uses a FIFO scheduler:

1. Open WebUI requests a model switch
2. llama-swap queues the switch request
3. Current streaming response completes
4. llama-swap tells llama.cpp to unload the old model and load the new one
5. New model is ready, queued request is served

Configuration:

```yaml
proxy: "http://localhost:8090"
routing:
  router:
    use: group
  swap: true
  exclusive: false
```

Model IDs in llama-swap must match the preset names in llama.cpp exactly.

## Why

Without llama-swap, model switches in llama.cpp abort in-flight streaming responses. The FIFO scheduler queues switches until the current stream drains. See [Orchestration Model](../../principles/orchestration-model.md) for the systemd-as-orchestrator pattern that keeps this running.
