# LXC containers

Containers rather than VMs for GPU workloads, so the GPU stays shareable
instead of being locked to a single guest. See ADR 0001.

## Planned

| Container | Purpose | GPU |
|---|---|---|
| `model-serving` (ID 200, created 2026-11-14) | Ollama / llama.cpp | 2070 Super |
| `gateway` (ID 201) | LiteLLM routing | none |
| `media` | Jellyfin or Plex (TBD) | possibly the 2070 Super (NVENC) for transcode |
| `sandbox` | Experiments — snapshot before every session | as needed |

## Convention

**IDs:** VMs in the 100s, containers in the 200s.

**Snapshot before experimenting.** The sandbox exists so that CUDA breakage is
a rollback, not an incident. This is the main reason the project runs a
hypervisor at all.
