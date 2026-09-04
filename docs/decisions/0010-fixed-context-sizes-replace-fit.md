---
title: "ADR-0010: Fixed Context Sizes"
---

# ADR-0010: Fixed context sizes replace `--fit` auto-fitting for llamacpp presets

## Status

Accepted

## Context

llamacpp presets were tuned with llama.cpp's `--fit` memory automation (`fit = on`, `fit-ctx`, `fit-target`). Two problems emerged:

1. **OpenHands cannot see the context size at all — with or without fit.**
   OpenHands Agent Canvas determines an LLM's context window from exactly two sources: an explicit `max_input_tokens` field on its LLM config, or litellm's static model registry keyed by name. It never queries llama.cpp's `/props` or `/v1/models`. For aliases like "Qwen 3.6 - Coding" every registry lookup fails, so `effective_max_input_tokens` stays unset. The proactive, token-pressure condensation path is therefore dead: compaction only fires via the 240-event heuristic or the reactive retry when the LLM raises a context-window error — which llama.cpp does not produce by default (it truncates/context-shifts instead).

2. **Fit-produced context sizes are load-time surprises, not contracts.**
   `fit-ctx` is a *minimum* floor for the fitter, not a set point. Worse, pinning any memory argument (e.g. `ngl = 99`) makes the whole fit pass abort — observed on the server as `common_fit_params: failed to fit params to free device memory: n_gpu_layers already set by user to 99, abort`, after which the context silently fell back to the configured floor.

llama.cpp reports the real context only *after* a model loads; unloaded presets expose none. A deterministic, explicit value per preset is the only thing downstream consumers can rely on.

## Decision

Replace fit-derived context sizing with explicit per-model context contracts in `playbooks/group_vars/home_server.yml` (`params.ctx_size`), rendered by the llamacpp template as `ctx-size` replacing the `fit-ctx`/`fit-target` blocks. All fit keys are removed from the `[*]` global section and from model params.

| Preset | ctx-size | Notes |
|--------|----------|-------|
| Gemma 4 - Text,Image | 262144 | uniform contract; ngl pin removed |
| Qwen 3.6 - Coding | 262144 | fit places layers around fixed `-c` |
| Qwen 3.8 - Text,Image | 262144 | native context window; fallback ladder if OOM: 196608 → 131072 |

Rationale per llama.cpp semantics: an explicit `--ctx-size` is never changed by `--fit`, so the fitter keeps managing GPU/RAM tensor placement around a known context.

OpenHands pairing step (manual): set the Qwen Coding LLM's max input tokens to ~245760 (262144 minus 16k output headroom). Headroom absorbs litellm-vs-llama.cpp tokenizer drift and reserves output room near the ceiling.

## Consequences

- **Positive:** Deterministic context budgets; OpenHands token-pressure condensation works once `max_input_tokens` is set; no silent fallback to the 8192 floor; reported `meta.n_ctx` matches config verbatim; uniform 262144 contracts across all production presets mean a single OpenHands `max_input_tokens` value (~245760) is valid regardless of loaded model.
- **Negative:** KV cache is always allocated at full contract size — oversized values either fail loudly at load (restart loop, when memory args are pinned) or degrade silently as fit sheds expert tensors to CPU (unpinned); VRAM can no longer flex down for co-tenant GPU workloads via fit-target.
- **Measured cost of uniform 256k on P40** (Gemma @ 262144 vs 131072, accepted 2026-08): token generation ~35 → ~31 t/s (−11%), prompt processing ~446 → ~81 t/s (−82%, PCIe bottleneck from CPU-resident experts). Fallback ladder if revisited: ctx 245760 → 196608 → 131072.

## Related

- ADR-0003 (ai-services pod)
