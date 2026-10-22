# Home Assistant

Runs as a **VM**, not a container — Home Assistant OS is the officially
supported install, and hard isolation is the point.

## Non-negotiable

Nothing in the ML/sandbox side of the machine may be able to take this down.
Home automation eventually controls lights, locks, and climate. It has to be
boring.

Phase 3 exit criteria: deliberately break the ML container and confirm the house
still responds.

## Integration

Calls the LiteLLM gateway (`infra/models/`) rather than any model endpoint
directly, so that a model being unavailable degrades gracefully.
