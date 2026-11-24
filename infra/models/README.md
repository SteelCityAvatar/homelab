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
| Version | **595.104.02** — installed 2026-11-11; verified with `nvidia-smi` and `dkms status` |
| CUDA | 13.2 is the newest this driver supports; container CUDA libraries must not be newer |
| Kernel module | Open (`Dual MIT/GPL`); uses GSP firmware `gsp_tu10x.bin` |
| Containers | Same version, installed with `--no-kernel-module` |
| Device files | `/dev/nvidia0`, `nvidiactl` (major 195, fixed), `nvidia-uvm`, `nvidia-uvm-tools` (major dynamic), mode 666. Created on first `nvidia-smi`, not at boot — boot trigger pending |

## Files

- `litellm-config.example.yaml` — starting sketch for the gateway. **Unverified.**
  Check it against current LiteLLM docs before use; the schema changes.

## Open

- Ollama vs llama.cpp — not decided
- Specific model choices — depends on what fits well in 8GB
