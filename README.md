```
           _
 _ __ _ __(_)_   ___ __  _ __
| '_ \| '__| \ \ / / '_ \| '_ \
| |_) | |  | |\ V /| |_) | | | |
| .__/|_|  |_| \_/ | .__/|_| |_|
|_|                 |_|
```

<p align="center">
  <img src="https://img.shields.io/badge/version-0.3.0-2a4f8c" alt="version 0.3.0">
  <img src="https://img.shields.io/badge/license-Apache%202.0-2a4f8c" alt="license: Apache 2.0">
</p>

**Your VPS. Your VPN. No third-party required.**

privpn turns any cheap Linux VPS into your personal WireGuard VPN — one script handles VPS access, install, peer management, and connect/disconnect. Nothing runs on someone else's "no-logs" promise. The server is yours, the keys are yours, the traffic is yours.

## Why

Commercial VPNs ask you to trust them. With privpn you trust yourself:

- **One-time setup, under 5 minutes** — interactive menu walks you through SSH key, install, peer configs.
- **All keys generated on the server** — nothing secret lives in this repo or your laptop until you connect.
- **Multi-device from day one** — phones get a QR code, Linux laptops auto-fetch their config over SSH, Macs get a paste-ready file.
- **No daemons, no GUI, no telemetry** — just `wg-quick` under the hood.

## Prerequisites

- **A VPS with a public IP.** Any Linux distro with a package manager (`apt` or `dnf`) — AlmaLinux, Ubuntu, Debian, Fedora, Rocky all work. ~2 EUR/month is plenty. Root or sudo access required. KVM-based VPSes are ideal; on VPSes where the WireGuard kernel module can't load, privpn falls back to userspace `wireguard-go` automatically.
- **WireGuard on your client.** `wireguard-tools` on Linux (`sudo dnf install wireguard-tools` / `sudo apt install wireguard`), the WireGuard app on iOS/Android/macOS.
- **SSH.** That's it. No `pass`, no `sshpass`, no Docker, no Python.

## Quick start

```bash
git clone https://github.com/hamr0/privpn.git && cd privpn
sudo ln -sf "$(pwd)/privpn" /usr/local/bin/privpn
privpn
```

First time: run option **1** (VPS access), then option **2** (Install WireGuard). After that: `privpn connect` / `privpn disconnect`.

## Menu

```
privpn — WireGuard VPN

  disconnected

  -- Setup (once) --
  1) VPS access             <- host, SSH key
  2) Install WireGuard      <- install / add / remove / QR

  -- VPN --
  3) Connect
  4) Disconnect
  5) Status

  -- Optional --
  6) Disable password login
  7) Save to pass
  0) Exit
```

Also works as a CLI: `privpn connect`, `privpn disconnect`, `privpn status`, `privpn setup`, `privpn wg`.

## How it works

```
Phone (anywhere)  ──┐
                    ├─ WireGuard tunnel (UDP 51820) ──> VPS (your IP)
Laptop (anywhere) ──┘                                    └─ NAT masquerade ──> Internet

VPN subnet: 10.100.0.0/24
Server:     10.100.0.1
Peers:      10.100.0.2, .3, .4, ...
```

1. **VPS access** — enter host/user/password once, copies your SSH key, never needs the password again.
2. **Install WireGuard** — installs on the VPS, generates all keys on the server, creates peer configs, prints a QR for phones.
3. **Connect** — fetches your config from the VPS via SSH, saves it locally, brings up the tunnel.
4. **Disconnect** — tears the tunnel down, re-enables IPv6.

## Files

| File | What |
|------|------|
| `privpn` | The CLI — setup, connect, disconnect, everything |
| `~/.config/privpn/config` | VPS host + username (created by setup) |
| `/etc/wireguard/privpn.conf` | Local WireGuard config (fetched from VPS) |

Keys and peer configs live on the VPS in `/etc/wireguard/`.

## Docs

| Doc | What |
|-----|------|
| [Customer Guide](customer-guide.md) | Full setup walkthrough, architecture, troubleshooting |
| [PRD](PRD.md) | What privpn is, principles, roadmap (incl. planned Docker mode) |
| [Changelog](CHANGELOG.md) | Version history |

## Related projects

privpn's niche is the **laptop-side orchestration over SSH** — it doesn't try to
re-do work others already do well. If you want a pure server-side installer or a
containerized WireGuard server, use these directly; privpn borrows ideas from
both rather than duplicating them:

- **[hwdsl2/wireguard-install](https://github.com/hwdsl2/wireguard-install)** — a
  mature, multi-distro WireGuard **installer + client manager you run on the VPS
  itself**. The reference for server-side robustness and peer add/remove.
- **[hwdsl2/docker-wireguard](https://github.com/hwdsl2/docker-wireguard)** — a
  **containerized** WireGuard server (docker-compose). The basis for privpn's
  planned optional Docker mode, and the source of the userspace `wireguard-go`
  fallback idea.

### Docker VPN servers (other protocols)

If privpn isn't the right fit, these other repos tackle the same goal — a
self-hosted, containerized VPN server — for different protocols. Reach for one of
them if you'd rather run OpenVPN, IPsec, or a self-hosted Tailscale control plane:

| Protocol | Project |
|----------|---------|
| WireGuard | [hwdsl2/docker-wireguard](https://github.com/hwdsl2/docker-wireguard) |
| OpenVPN | [hwdsl2/docker-openvpn](https://github.com/hwdsl2/docker-openvpn) |
| IPsec VPN | [hwdsl2/docker-ipsec-vpn-server](https://github.com/hwdsl2/docker-ipsec-vpn-server) |
| Headscale | [hwdsl2/docker-headscale](https://github.com/hwdsl2/docker-headscale) |

## License

Apache 2.0 — see [LICENSE](LICENSE).
