# privpn — Product Requirements

> Living document. Records what privpn is, the principles it holds to, and what's
> planned next. Update alongside `CHANGELOG.md` when scope changes.

## Problem

Commercial VPNs require trusting a third party's "no-logs" promise. People who
own a cheap Linux VPS can host their own WireGuard VPN instead — but the setup
(SSH hardening, key generation, peer configs, firewall, connect/disconnect) is
fiddly and easy to get wrong. privpn makes that a sub-5-minute, repeatable flow.

## What privpn is

A single, dependency-free bash CLI that runs **on your laptop** and orchestrates
your VPS **over SSH**. It is deliberately not a server-side installer you copy onto
the box — that niche is already well covered (see [Related projects](#related-projects)).
privpn's value is the local-side flow: key-copy, install, peer management, and
`connect` / `disconnect` against a dedicated `privpn` interface, mirroring the
step-based setup pattern of the sibling `privcloud` project.

## Principles

- **Existing working code is the spec.** The privcloud step-based pattern is the
  template; don't diverge from it without reason.
- **Idempotent.** Every step is safe to re-run; setup detects existing state.
- **Verify real outcomes, not tool execution.** Confirm a backend actually works,
  confirm the tunnel is actually up — don't claim success on a clean exit code.
- **No daemons, no telemetry, no extra deps** beyond `ssh` and `wg-quick`.
- **Keys are generated on the server**, never committed, never on the laptop until
  connect.

## Current state (v0.3.0)

- VPS access setup: SSH keygen + `ssh-copy-id`, idempotent, key-auth verified.
- WireGuard install over SSH: dnf/apt, server + peer key generation on the VPS,
  IP forwarding, firewall (firewalld/ufw), NAT masquerade via iptables.
- **Backend detection with userspace fallback**: uses the kernel module when it
  loads, falls back to `wireguard-go` when it doesn't, fails loudly when neither
  is available.
- Peer management: add, **remove**, show config / QR, reinstall.
- Connect / disconnect / status against the dedicated `privpn` interface, with
  IPv6 disabled while the tunnel is up (leak prevention — IPv4-only by design).
- Optional: disable VPS password login, back everything up to `pass`.

## Roadmap

### Next: optional Docker deployment for the VPS side

Goal: let the VPS run WireGuard as a container instead of bare-metal `wg-quick`,
so the VPS side matches the `privcloud` house style (Docker Compose + single `.env`).

- Base it on [`hwdsl2/docker-wireguard`](https://github.com/hwdsl2/docker-wireguard)
  rather than building an image — it already handles kernel/userspace selection,
  key generation, QR, and persistence.
- privpn would: ensure Docker is present on the VPS, drop a `docker-compose.yml` +
  `.env`, `docker compose up`, and manage peers via `docker exec ... wg_manage`
  instead of editing `/etc/wireguard/*.conf` directly.
- **Open decision:** this changes the peer-management surface (file edits →
  `wg_manage`) and the connect-side config fetch. Treat as a parallel "mode," not
  a replacement, unless bare-metal is dropped entirely.
- **Effort estimate:** the image itself is easy (compose file + `.env`, container
  self-configures on first launch). The real work is the privpn integration:
  detect/install Docker on the VPS, swap the install + peer paths, and keep
  `connect` fetching the right client config. Moderate, not large.

### Later / backlog

- `listclients` / status of peers as a first-class menu item.
- Per-peer DNS override.
- IPv6 split-tunnel option (currently IPv4-only by deliberate choice to avoid leaks).

## Non-goals

- Being a general server installer or client manager run *on* the VPS — use the
  referenced projects for that.
- GUI, daemon, or telemetry.
- Commercial multi-tenant / many-user management.

## Related projects

privpn does not duplicate work others have already done well. For the
server-/container-side, these are the references it borrows ideas from:

- [`hwdsl2/wireguard-install`](https://github.com/hwdsl2/wireguard-install) —
  battle-tested server-side installer + client manager. Reference for distro
  robustness and peer add/remove.
- [`hwdsl2/docker-wireguard`](https://github.com/hwdsl2/docker-wireguard) —
  containerized WireGuard server. Reference for the userspace `wireguard-go`
  fallback and the planned Docker mode.
