# 0008 — Remote access through Tailscale; no inbound port forwards

**Status:** Accepted
**Date:** 2026-11-24

## Context

The server needs to be reachable while away from home: SSH for administration, the
Proxmox web UI on `:8006`, and later model endpoints and Home Assistant. Both
addresses (`192.168.1.169`, `192.168.1.164`) are private and unreachable from
outside.

Options:

- **Port forwarding** on the router (22, 8006, …) straight to the server.
- **A self-hosted WireGuard** endpoint, which still needs one forwarded UDP port and
  manual key distribution.
- **Tailscale**: a WireGuard-based mesh VPN; each device connects outward to a
  coordination service and peers connect directly.

## Decision

- Install **Tailscale on the Proxmox host** (`steel`) and on each client device.
  Administer remotely over the tailnet only.
- **No inbound port forwards** on the router, for any service.
- Keep **SSH key authentication** as the login control; Tailscale is the network
  path, not the credential. `tailscale up` runs without `--ssh`.

## Rationale

- **Nothing exposed.** No open port for internet-wide scanners to find. The
  alternative puts a root login on the public internet.
- **Works through any NAT.** Connections are outbound; no router changes, works on
  hotel and mobile networks (relay fallback when a direct path fails).
- **Stable addresses.** Tailscale addresses (`100.64.0.0/10`) and MagicDNS names
  don't depend on the router's DHCP.
- **Low effort.** One package per device; updates via apt on the host.

## Consequences

- Access depends on the Tailscale account: protect its login with two-factor.
- The coordination service is a third party. It brokers connections but does not
  carry or decrypt traffic.
- **Disable key expiry for `steel`** in the admin console, or the server drops off
  the tailnet after 180 days.
- SSH user keys carry a `from=` restriction to the LAN and `100.64.0.0/10`.
- Remote access is not remote recovery: avoid host reboots and unattended load tests
  while away; set the BIOS to power on after AC loss.
- Home Assistant (VM) gets remote access the same way (its Tailscale add-on, or the
  host as a subnet router) — still no `:8123` port forward.
