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

- [ ] **PSU re-test under GPU load** — `bom.md` #23 condition (c). The TX750M is
      ~13 years old; Phase 2 is the first sustained GPU load.
- [ ] ⚠️ **Decide about the GTX 1060.** NVIDIA's **580 driver branch is the last to
      support Pascal** (GTX 10-series); newer branches and CUDA 13 drop it. One host
      runs **one** driver for both cards, so keeping the 1060 pins the whole server to
      the 580 legacy branch — which may lag behind new Proxmox kernels.
      Options: (a) 580 branch for both, (b) **pull the 1060**, run current drivers
      for the 2070 Super alone, (c) keep it installed but unused.
      → Leaning (b) unless the 1060's job (Whisper/embeddings) is needed early.
- [ ] Decide the driver source: Debian's packaged driver vs NVIDIA's `.run`
      installer. ⚠️ The container's userspace driver **must match the host's
      version exactly**; the `.run` route makes that easy to control.

## Phase 1 — NVIDIA driver on the host

- [ ] Install build tools + kernel headers for the **running** kernel
      (⚠️ package name on PVE 9: `proxmox-headers-$(uname -r)` or similar)
- [ ] Blacklist `nouveau` (the open-source driver), rebuild initramfs, reboot
- [ ] Install the chosen driver
- [ ] Verify: `nvidia-smi` lists the GPU(s), driver version, temps, idle power
      → also fixes the "GPU idles warm with no driver" note
- [ ] Make sure `/dev/nvidia-uvm*` exist at boot (⚠️ may need `nvidia-persistenced`
      or a udev/modprobe rule)
- [ ] **Kernel-update caveat:** every new Proxmox kernel needs the module rebuilt.
      DKMS automates it; the `.run` installer with DKMS does too. Test once:
      reboot after a kernel update and check `nvidia-smi`.

## Phase 2 — Model-serving LXC

- [ ] Create an **unprivileged** LXC from the Debian 13 template: ~4 cores, 8 GB
      RAM, disk sized for models (⚠️ a 7–8B Q4 model is ~5 GB; plan 100+ GB)
- [ ] Pass the GPU devices in: Resources → Add → **Device Passthrough**
      (`/dev/nvidia0`, `/dev/nvidiactl`, `/dev/nvidia-uvm`, `/dev/nvidia-uvm-tools`)
- [ ] Install the **same driver version** inside the container, userspace only
      (no kernel module)
- [ ] Verify `nvidia-smi` works **inside** the container
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

- [ ] Only if the 1060 stays: Whisper / embeddings pinned to it with
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
| 1060 forces a legacy driver branch | Decide in Phase 0; pulling the card is the simple path |
| Old PSU under sustained GPU load | Re-test first (`bom.md` #23 c) |
| 8 GB VRAM too small | Stay at 7–8B Q4; escalate hard tasks to the 4090 / API |
