# homelab

A dedicated, always-on home server, built from scratch, for **storage, local AI
inference, and home automation**. This repo documents the build: hardware,
architecture, decisions, and runbooks.

It's a new machine with its own job. My gaming/home-theater desktop used to
pull double duty as "the server". That works until you reboot for a game and
everything that depended on it goes down. This build splits the roles:
the server stays up and holds state, and the desktop goes back to being a
desktop that occasionally lends its GPU.

| | Node A — Desktop | Node B — Server (this build) |
|---|---|---|
| Role | Gaming, home theater, burst GPU compute | Always-on storage, AI inference, automation |
| Hardware | i7-13700K · RTX 4090 24GB · 32GB DDR5 | i7-8700K · RTX 2080 Ti 11GB + GTX 1060 6GB · 32GB DDR4 |
| OS | Windows 11 | Proxmox VE |
| Availability | Low (gaming, reboots) | High (always on) |
| Status | Existing, unchanged | **Building — Phase 0 (hardware verification)** |

## What it's for

**1. AI utility: local model serving with tiered fallback**
- An always-on 7-8B model on the 2080 Ti handles routing, tool selection,
  classification, and Home Assistant calls
- The GTX 1060 runs specialized work: Whisper transcription, embeddings for RAG
- A LiteLLM gateway escalates hard requests to a 27-32B model on the desktop's
  4090 when it's free, and to a frontier API when it isn't
- Launching a game makes the desktop's health check fail, so the system stops
  escalating instead of hanging. It degrades; it doesn't break.

**2. Storage: tiered by how replaceable the data is**
- ZFS mirror for irreplaceable data (photos, documents, code)
- Unmirrored bulk tier for re-acquirable media
- Off-server backup before anything leaves the desktop

**3. Home automation that never goes down**
- Home Assistant OS in an isolated VM. Breaking an ML container can't take
  down the house.

**4. Sandbox and services**
- Snapshot-before-every-experiment LXC sandbox
- Media server (Jellyfin or Plex, TBD)

## Architecture at a glance

```
Proxmox VE (bare metal, Node B)
├── ZFS pools (Vault mirror + Bulk)
├── VM:  Home Assistant OS        ← isolated, must never break
├── LXC: model-serving            ← GPU passthrough, Ollama / llama.cpp
├── LXC: gateway                  ← LiteLLM routing + fallback chain
├── LXC: media                    ← Jellyfin / Plex
└── LXC: sandbox                  ← snapshot before every experiment

request → LiteLLM (server) ─┬─ simple → 7-8B on 2080 Ti
                            └─ hard   → 4090 on desktop if alive
                                        └─ else queue / frontier API
```

Full detail in [`docs/architecture.md`](docs/architecture.md).

## Design principles

- **The always-on node holds coordination and state.** The high-power node is
  an opportunistic worker, never the reverse ([ADR 0003](docs/decisions/0003-orchestrator-on-server.md)).
- **Work within real constraints.** 4 drive bays, 32GB RAM, 11GB VRAM, and a
  750W PSU are the ceilings, and the design is sized to them.
- **Mismatched GPUs are two workers, not one pool.** Turing + Pascal don't pool
  VRAM, so each gets its own job.
- **Decisions are written down.** Every significant choice has an ADR in
  [`docs/decisions/`](docs/decisions/). Anything unverified is marked `TBD` / ⚠️.

## Roadmap

| Phase | Goal | Status |
|---|---|---|
| 0 | Verify hardware (POST, RAM, PSU, bays, placement) | **In progress** |
| 1 | Storage foundation: Proxmox, ZFS mirror, data migration, backup | Drives on hand |
| 2 | Model serving: GPU LXCs, Ollama, LiteLLM fallback chain | Planned |
| 3 | Home automation: Home Assistant VM + gateway integration | Planned |
| 4 | Polish: UPS, 2.5GbE node-to-node link, media server | Planned |

Details and checklists in [`ROADMAP.md`](ROADMAP.md).

## Repo map

```
ROADMAP.md             Phased build plan
bom.md                 Bill of materials / purchase tracker
docs/
  spec.md              Hardware spec sheet
  architecture.md      System + model topology
  decisions/           ADRs: why things are the way they are
  runbooks/            Step-by-step operational procedures
infra/
  proxmox/             Host config
  lxc/                 Container definitions (ML/model serving)
  models/              Inference + routing config
  homeassistant/       HA VM config
CLAUDE.md, HANDOFF.md  Working context for AI-assisted build sessions
```
