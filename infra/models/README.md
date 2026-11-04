# Model serving

Runs in an LXC container with GPU device passthrough from the Proxmox host.

## Layout

| GPU | Workload |
|---|---|
| RTX 2070 Super (8GB) | Local model — routing, tool selection, classification |

One GPU. The GTX 1060 is out of the build (ADR 0007); a second lane for Whisper /
embeddings is optional later.

## Driver

| Item | Value |
|---|---|
| Source | NVIDIA `.run` installer, DKMS (ADR 0007) |
| Version | **595.104.02** — ⚠️ not yet installed (2026-11-04) |
| Containers | Same version, installed with `--no-kernel-module` |

## Files

- `litellm-config.example.yaml` — starting sketch for the gateway. **Unverified.**
  Check it against current LiteLLM docs before use; the schema changes.

## Open

- Ollama vs llama.cpp — not decided
- Specific model choices — depends on what fits well in 8GB
