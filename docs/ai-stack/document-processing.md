---
title: Document Processing
---

# Document Processing

## What

Docling-serve converts documents (PDFs, images, text files) into structured data for Open WebUI's RAG pipeline. It runs as a CPU-only container.

## How

The container runs `quay.io/docling-project/docling-serve-cpu` — the lightweight variant without LibreOffice or ffmpeg.

Key configuration:

| Setting | Value | Reason |
|---------|-------|--------|
| `DOCLING_SERVE_MAX_SYNC_WAIT` | 600 | Supports large PDFs (13 MB+) |
| `DOCLING_SERVE_ENG_LOC_NUM_WORKERS` | 4 | Conversion throughput |
| `UVICORN_WORKERS` | 1 | Multiple workers cause "Task Not Found" errors |

Open WebUI is configured to use docling as its extraction engine:

```ini
Environment=CONTENT_EXTRACTION_ENGINE=docling
```

## Why

Docling was chosen over alternatives (Unstructured, LlamaParse) because:

- Self-hosted — no data leaves the network
- CPU-only image available — no GPU required for document processing
- Good PDF support — handles tables, images, and complex layouts
- Active development — the docling project is actively maintained

The `UVICORN_WORKERS=1` setting is a gotcha. Multiple workers cause "Task Not Found" errors with async conversion requests. This is a docling-serve limitation, not a configuration error. The docs recommend staying at 1 worker.

!!! note "Supported file types"
    The docling-serve-cpu image supports PDF, images, and plain text. Legacy Office formats (`.doc`, `.xls`, `.ppt`) require LibreOffice (not included). Audio/video require ffmpeg (not included). See [ADR-0006](../decisions/0006-oikb-docling-filter.md) for the file type allowlist.
