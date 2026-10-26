# 0003 — Orchestrator model on the server, not the desktop

**Status:** Accepted
**Date:** 2026-08-09

## Context

The original proposal was to run a large "master" orchestrator model on the
desktop's RTX 4090 (24GB) and weaker companion models on the server. The 4090
is by far the more capable card.

## Decision

The orchestrator/router runs on the **server**. The desktop's 4090 is an
opportunistic escalation backend.

## Rationale

The coordinator is the component that must be up for *anything* to work —
workers are inert without it. Putting it on the desktop makes total system
uptime equal to the uptime of a Windows gaming machine that sleeps, reboots for
updates, and is in active use most evenings.

The VRAM collision makes it concrete: a 27-32B model at Q4 wants ~18GB, a modern
game wants 12-20GB, and they do not coexist on a 24GB card. The orchestrator
would unload exactly when the user sits down on the couch — taking home
automation and scheduled jobs with it.

Routing, tool selection, and classification are constrained tasks that small
models handle acceptably. Hard reasoning is what gets escalated.

## Consequences

- Router quality is capped by what fits in 8GB (~7-8B at Q4)
- Requires a health-check + fallback chain (LiteLLM) rather than a static route
- System degrades gracefully when the desktop is gaming instead of hanging

> **Correction 2026-10-26:** the server GPU is an RTX 2070 Super (8GB, ~215W),
> not a 2080 Ti (11GB, ~260W). Figures above updated; the decision is unchanged.
