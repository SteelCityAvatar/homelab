# Runbook — working on `steel` remotely

Remote access is over Tailscale only (ADR 0008). No ports are forwarded on the router.

## Connect

| From | Do |
|---|---|
| Any device | Tailscale app installed, signed in to the same tailnet account, toggled **on** |
| Web UI | `https://steel:8006` (MagicDNS). Fallback: the host's `100.x` address from `tailscale status` or the admin console. Accept the self-signed certificate warning |
| SSH | `ssh root@steel`. Needs that device's own key in `/root/.ssh/authorized_keys` with the `from=` limit (see `infra/proxmox/README.md`) |
| Phone, no SSH client | Web UI → `steel` → **>_ Shell** gives a root shell in the browser |

**Check the path:** `tailscale ping steel` → "via DERP" means relayed (slower, still
encrypted); an IP:port means direct.

## Safe vs on-site only

| Safe remotely | On-site only |
|---|---|
| Reading logs, `nvidia-smi`, `tailscale status`, health checks | **Host reboots** (a hung boot can't be fixed remotely) |
| Container start/stop/restart, config changes | **PSU load test** / sustained GPU load |
| GPU passthrough config for CT 200, driver userspace inside it | BIOS changes, hardware |
| `apt update` (list only) | `apt full-upgrade` when it brings a **new kernel** (needs a reboot; DKMS must rebuild) |
| Editing files, installing packages in containers | Anything touching networking on the host (`/etc/network/interfaces`, `vmbr0`) — a mistake cuts off access |

Run anything long inside `tmux` (`tmux new -s work`, reattach with `tmux attach -t work`)
so a dropped connection doesn't kill it.

## If something goes wrong

| Symptom | Likely cause | Action |
|---|---|---|
| `steel` offline in Tailscale, web UI unreachable | Host down, power cut, or ISP outage | Nothing remote can fix it. With "restore on AC power loss" set, a power cut self-recovers. Wait, or ask someone on site to press power |
| Tailscale offline but nothing else changed | Node key expired, or `tailscaled` stopped | Key expiry should be disabled for `steel`; check the admin console |
| SSH "Permission denied (publickey)" | This device's key isn't authorized, or connecting from outside `from=` ranges | Use the web UI shell instead; add the key from there |
| Container has no GPU after an unexpected host restart | The boot trigger (`70-nvidia.rules`) didn't fire — unproven as of 2026-11-24 | Run `nvidia-smi` once on the host, then restart CT 200 |
| Root filesystem read-only (writes fail everywhere) | NVMe dropout — see the 2026-11-01 incident | Don't reboot remotely if avoidable; capture `dmesg` first. Needs on-site recovery |
