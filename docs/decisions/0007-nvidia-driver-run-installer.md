# 0007 — NVIDIA driver from NVIDIA's `.run` installer, with DKMS; 2070 Super only

**Status:** Accepted
**Date:** 2026-11-03

## Context

Model serving runs in LXC containers that share the host's GPU (ADR 0001). A
container uses the host's kernel module but needs its own copy of the driver's
userspace libraries — and **the two must be the exact same version**, or the GPU
fails inside the container.

The GTX 1060 (Pascal) is not installed. NVIDIA's 580 branch is the last to support
Pascal; keeping it would pin the host to a legacy branch for both cards.

Options for the driver: Debian's packaged `nvidia-driver`, or NVIDIA's `.run`
installer.

## Decision

- Install the driver from **NVIDIA's `.run` installer**, built with **DKMS**, on the
  Proxmox host; install the **same version** in containers with
  `--no-kernel-module`.
- Run the **RTX 2070 Super alone**, on a current driver branch. The 1060 stays out.

## Rationale

- **Version control.** With `.run`, the version is chosen explicitly and installed
  identically on host and containers. Debian's package updates via `apt` on its own
  schedule and can leave containers on a mismatched version.
- **Kernel updates.** DKMS rebuilds the module automatically for each new Proxmox
  kernel.
- **One card, current driver.** Avoids legacy-branch lag behind new kernels.

## Consequences

- Driver upgrades are manual and must be repeated in every GPU container.
- Record the installed version in `infra/models/README.md`.
- After each kernel update, confirm `nvidia-smi` still works before relying on the GPU.
- Revisit the 1060 only if a second GPU lane (Whisper / embeddings) is needed.
