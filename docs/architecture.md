# Architecture

## Two-node model

```
┌──────────────────────────┐         ┌──────────────────────────┐
│  NODE A — Desktop        │         │  NODE B — Server         │
│  Windows 11              │         │  Proxmox VE              │
│                          │         │                          │
│  RTX 4090 (24GB)         │◄───────►│  RTX 2080 Ti (11GB)      │
│  13700K / 32GB DDR5      │  LAN    │  GTX 1060 (6GB)          │
│  9TB game+scratch        │         │  8700K / 32GB DDR4       │
│                          │         │  ZFS pools               │
│  ALSO: home theater      │         │                          │
│  (S90D OLED + 5.1)       │         │  ALWAYS ON               │
│                          │         │                          │
│  Availability: LOW       │         │  Availability: HIGH      │
│  (gaming, reboots)       │         │                          │
└──────────────────────────┘         └──────────────────────────┘
```

**Design principle:** the always-on node holds coordination and state. The
high-power node is an opportunistic worker. Never the reverse — putting the
orchestrator on the machine that reboots for games makes total system uptime
equal to the least-available node. See ADR 0003.

---

## Virtualization layout (Node B)

```
Proxmox VE (bare metal)
├── NVIDIA driver on host
├── ZFS pools
│
├── VM: Home Assistant OS          ← isolated, must never break
│
├── LXC: model-serving             ← GPU device passthrough
│     └── Ollama / llama.cpp
├── LXC: gateway
│     └── LiteLLM (routing + fallbacks)
├── LXC: media
│     └── Jellyfin or Plex (TBD)
└── LXC: sandbox                   ← snapshot before every experiment
```

**Why LXC and not VMs for GPU work:** containers share a GPU with the host.
VM passthrough dedicates the card to one guest, which would lock the 2080 Ti
away from everything else.

**Why HA in a VM and not LXC:** Home Assistant OS is the officially supported
install, and hard isolation is the point — experiments must not be able to take
down home automation.

---

## Storage tiers

Split by **replaceability**, not one uniform redundancy scheme. Mirroring media
that could be re-acquired wastes money during a shortage.

| Tier | Contents | Redundancy | Media |
|---|---|---|---|
| Boot | Proxmox OS | None (rebuildable) | NVMe (M2_1) |
| Hot | Model weights, datasets, VM/LXC disks | None | NVMe (M2_2) |
| Vault | Photos, documents, code — irreplaceable | ZFS mirror | 2x HDD |
| Bulk | Media library — re-acquirable | None | 2x HDD |

Open: capacity per drive, current data volume (~5TB?), Vault backup target,
offsite / 3-2-1 strategy.

---

## Model topology

| Node | Model class | Availability | Job |
|---|---|---|---|
| Server / 2080 Ti | 7-8B, Q4-Q5 | Always on | Routing, tool selection, classification, HA calls |
| Server / 1060 | Small / specialized | Always on | Whisper, embeddings for RAG |
| Desktop / 4090 | 27-32B, Q4 | Opportunistic | Hard reasoning, code generation |
| External API | Frontier | On demand | Escalation for hard bespoke-code work |

### Fallback chain

```
request
   │
   ▼
LiteLLM gateway (server, always up)
   │
   ├── simple? ───────────► local 7-8B on 2080 Ti
   │
   └── hard? ──► health-check desktop endpoint
                    │
                    ├── alive ──► 27-32B on 4090
                    └── gaming/asleep ──► queue, or fall through to API
```

Launching a game unloads the desktop model, the health check fails, and the
system stops escalating rather than hanging. Degradation, not breakage.

### Known limits

- 11GB VRAM caps the server at ~7-8B comfortably (14B if squeezed)
- The two server GPUs **will not pool**. Mismatched generations, mismatched
  memory. Two independent workers, not 17GB.
- Small models are decent at constrained routing (pick a tool, emit JSON) but
  flaky at multi-step planning. If the 8B loses the thread, narrow its job
  rather than upgrading it.
- A local 32B will not match a frontier API for hard code generation. Local
  earns its keep on privacy-sensitive, high-volume, and offline work.

---

## Networking

Onboard is 1GbE (Realtek RTL8111H). For bulk transfers between nodes, add a
2.5GbE NIC to each machine and run them **directly to each other** on a separate
subnet — no switch purchase needed. Normal LAN traffic keeps using gigabit.
