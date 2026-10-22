# 0001 — Proxmox VE over Unraid or bare Ubuntu

**Status:** Accepted
**Date:** 2026-08-09

## Context

Three stated priorities: bulk storage, a sandbox for trying models, and home
automation. Candidate platforms were Unraid (storage-first), bare Ubuntu +
Docker (simplest CUDA story), and Proxmox VE (hypervisor).

## Decision

Proxmox VE on bare metal, with ZFS.

## Rationale

The priorities have **conflicting reliability requirements**. Home Assistant
will eventually control lights, locks, and climate — it must be boring and never
go down. A model sandbox is where things get deliberately broken: mismatched
CUDA versions, half-working drivers, containers that wedge a GPU.

On one flat OS, every experiment is one bad install away from the house not
responding. A hypervisor gives hard isolation plus snapshot-and-rollback, which
is exactly the property a sandbox wants.

Bare Ubuntu has the easiest CUDA story but no isolation and no snapshots.
Unraid costs money and its main advantage — mixing arbitrary drive sizes — is
less relevant once the plan settled on matched mirrored pairs.

## Consequences

- GPU access to containers requires host driver + LXC device passthrough rather
  than the simpler bare-metal `nvidia-container-toolkit` path
- ZFS ARC must be capped given 32GB total RAM
- Snapshot-before-experiment becomes a standard workflow
