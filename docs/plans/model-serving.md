# Plan: Model serving (NVIDIA driver + Ollama LXC + gateway)

**Goal:** an always-on local model on the RTX 2070 Super, reachable on the LAN
through a LiteLLM gateway that escalates to the desktop's 4090 when it's free
(ADR 0003, ROADMAP Phase 2).

**Type:** LXC containers sharing the host's GPU driver. Containers share one card;
a VM would lock it to one guest (ADR 0001).

**Depends on:** the NVIDIA driver on the Proxmox host — the hardest step of the
build. Items marked ⚠️ get verified at the time, not assumed.

---

## Phase 0 — Gates and decisions (before touching drivers)

- [ ] **PSU re-test under GPU load** — `bom.md` #23 condition (c). Needs the driver
      and a CUDA program to create the load, so it runs **inside the model-serving
      LXC, before Ollama** (Phase 2). Decided 2026-11-11: keeps CUDA libraries off
      the host. Creating the container puts no load on the GPU.
- [x] **Decide about the GTX 1060.** NVIDIA's **580 driver branch is the last to
      support Pascal** (GTX 10-series); newer branches and CUDA 13 drop it. One host
      runs **one** driver for both cards, so keeping the 1060 pins the whole server to
      the 580 legacy branch — which may lag behind new Proxmox kernels.
      Options: (a) 580 branch for both, (b) **pull the 1060**, run current drivers
      for the 2070 Super alone, (c) keep it installed but unused.
      → **Decided (b), 2026-11-03** — the 1060 isn't installed. ADR 0007.
- [x] Driver source: **NVIDIA's `.run` installer with DKMS** — decided 2026-11-03,
      ADR 0007. Host and containers run the exact same version.

## Phase 1 — NVIDIA driver on the host

- [x] Install build tools + kernel headers — `proxmox-default-headers`,
      `proxmox-headers-$(uname -r)`, `build-essential`, `dkms` (2026-11-03)
- [x] Blacklist `nouveau`, rebuild initramfs, reboot — nouveau no longer loads (2026-11-03)
- [x] Install the driver — **595.104.02** (`.run`, DKMS), 2026-11-11. The first attempt
      (2026-11-09) was killed by a dropped SSH session: run installers inside `tmux`
- [x] Verify: `nvidia-smi` → RTX 2070 SUPER, 595.104.02, CUDA 13.2; idle P8, 16 W,
      28 °C (2026-11-11). `dkms status` → installed for 7.0.14-26-pve
- [ ] Make sure `/dev/nvidia*` exist at boot. 2026-11-22: all four present
      (`nvidia0`, `nvidiactl`, `nvidia-uvm`, `nvidia-uvm-tools`), but created on demand
      by the first `nvidia-smi`, not at boot; no `nvidia-persistenced`, no NVIDIA
      udev/modules-load rules. Fix: `/etc/modules-load.d/nvidia.conf` (`nvidia`,
      `nvidia_uvm`) + a udev rule running `nvidia-smi` / `nvidia-modprobe -c0 -u`.
      Needs a host reboot to verify — **deferred until on site** (ADR 0008
      consequences). Neither file exists yet (checked 2026-11-24).
      Major 195 is fixed; uvm/caps majors are dynamic → pass devices by path
- PSU load test → moved into Phase 2, before Ollama (2026-11-11)
- [ ] **Kernel-update caveat:** every new Proxmox kernel needs the module rebuilt.
      DKMS automates it; the `.run` installer with DKMS does too. Test once:
      reboot after a kernel update and check `nvidia-smi`.

## Phase 2 — Model-serving LXC

- [x] Create an **unprivileged** LXC from the Debian 13 template — ID **200**,
      `model-serving`: 4 cores, 8 GB RAM, 100 GB thin disk, nesting on (2026-11-14).
      On the LAN at `192.168.1.164/24` by DHCP (2026-11-22). ⚠️ Not reserved — reserve
      it on the router before Ollama goes live
- [ ] Pass the GPU devices in: Resources → Add → **Device Passthrough**
      (`/dev/nvidia0`, `/dev/nvidiactl`, `/dev/nvidia-uvm`, `/dev/nvidia-uvm-tools`)
- [ ] Install the **same driver version** inside the container, userspace only
      (no kernel module)
- [ ] Verify `nvidia-smi` works **inside** the container
- [ ] **PSU load test — gates Ollama:** ~30 min of large matrix multiplies (PyTorch) in
      one tmux pane, `nvidia-smi` in another, listening to the PSU. Any reboot or
      shutdown under load → replace the PSU (`bom.md` #23 b)
- [ ] Install Ollama; bind it to the LAN (`OLLAMA_HOST=0.0.0.0`)
- [ ] Pull a 7–8B model at Q4 (8 GB VRAM ceiling); test a prompt; watch VRAM in
      `nvidia-smi`
- [ ] Snapshot the container once it works — rollback point before experiments

## Phase 3 — Desktop endpoint (Node A)

- [ ] Ollama on Windows, bound to the LAN, serving a 27–32B model on the 4090
- [ ] Windows firewall rule for its port, LAN only

## Phase 4 — Gateway LXC

- [ ] Small LXC (1 core, 1 GB) running LiteLLM
- [ ] Config: local router model → desktop model → API fallback
      (`infra/models/litellm-config.example.yaml` — ⚠️ verify against current
      LiteLLM docs, it's a sketch)
- [ ] **Graceful-degradation test:** start a game on the desktop, confirm requests
      fall back instead of hanging

## Phase 5 — Optional: second GPU lane

- [ ] Only if a second GPU is added: Whisper / embeddings pinned to it with
      `CUDA_VISIBLE_DEVICES`

## Done when

A request to the gateway is answered by the 2070 Super, escalates to the 4090 when
the desktop is free, falls back cleanly when it isn't — and a kernel update plus
reboot doesn't break the GPU.

## Risks

| Risk | Mitigation |
|---|---|
| Kernel update breaks the driver | DKMS; test after each kernel update; keep the previous kernel bootable |
| Host / container driver mismatch | Same installer version in both; record it in `infra/models/README.md` |
| 1060 forces a legacy driver branch | Resolved — the 1060 is out (ADR 0007) |
| Old PSU under sustained GPU load | Re-test first (`bom.md` #23 c) |
| 8 GB VRAM too small | Stay at 7–8B Q4; escalate hard tasks to the 4090 / API |
