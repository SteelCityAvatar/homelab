# Model serving

Runs in an LXC container with GPU device passthrough from the Proxmox host.

## Layout

| GPU | Workload |
|---|---|
| RTX 2070 Super (8GB) | Primary local model — routing, tool selection, classification |
| GTX 1060 (6GB) | Secondary lane — Whisper, embeddings for RAG |

The GPUs are pinned to separate workloads via `CUDA_VISIBLE_DEVICES`. They are
**not** pooled — see `docs/architecture.md`.

## Files

- `litellm-config.example.yaml` — starting sketch for the gateway. **Unverified.**
  Check it against current LiteLLM docs before use; the schema changes.

## Open

- Ollama vs llama.cpp — not decided
- Specific model choices — depends on what fits well in 8GB
