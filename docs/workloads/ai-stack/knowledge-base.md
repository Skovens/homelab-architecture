---
title: Knowledge Base
---

# Knowledge Base

oikb syncs external sources into Open WebUI Knowledge Bases for RAG (Retrieval-Augmented Generation). Currently it syncs the Obsidian vault.

## How

oikb runs on port 8084 and syncs the Obsidian vault from a local git clone. The `forgejo-vault-sync` systemd service keeps the clone current with bidirectional sync:

1. **Pull** remote changes from Forgejo (`git pull --rebase --autostash`)
2. **Commit** any AI-authored vault-mcp writes (author: `open-webui-ai`)
3. **Push** back to Forgejo with retry on rejection
4. **Trigger** oikb KB re-index only when HEAD moves (avoids re-upload loop)

File type filtering restricts sync to docling-supported formats:

```yaml
oikb_docling_filter: &oikb_docling_filter
  include_extensions:
    - ".pdf"
    - ".md"
    - ".txt"
    - ".png"
    - ".jpg"
    - ".jpeg"
    - ".csv"
    - ".json"
```

The YAML anchor (`&oikb_docling_filter`) is shared across all sources — adding a new source automatically inherits the filter.

## Why

The file type filter restricts sync to docling-supported formats. Without it, unsupported files would trigger conversion failures on every sync cycle.
