# Changelog

## 2026-04-06 — Save to pass: full backup of all configs and keys

- Save to pass now stores everything: VPS host/user, SSH keys, all WireGuard configs
- Fetches server config and all peer configs from VPS
- Overwrites existing entries (no prompts, idempotent)
- Shows restore commands after saving

## 2026-04-05 — Full rewrite: SSH key auth, keys on server, no pass dependency

**Breaking change:** old `pass`-based setup no longer works. Run fresh setup.

- Rewrote `privpn` CLI with step-based setup matching privcloud pattern
- VPS access via SSH key auth (copies key on first setup, no more `sshpass`)
- WireGuard keys generated on the VPS (not locally, not in `pass`)
- Peer configs stored on VPS (`/etc/wireguard/<name>.conf`)
- `privpn connect` fetches config from VPS automatically via SSH
- Dedicated `privpn` WireGuard interface (won't conflict with fedvpn/wg0)
- Config saved to `/etc/wireguard/privpn.conf` (persistent, not `/tmp/`)
- Supports dnf and apt (works on any VPS, not just AlmaLinux)
- Optional: disable password login with lockout warning
- Optional: save SSH key + config to `pass` as backup
- Menu separated into Setup / VPN / Optional sections
- Deleted legacy scripts: `setup.sh`, `vpn-up.sh`, `vpn-down.sh`, `qr.sh`, `status.sh`
- Deleted outdated docs: `vps_setup.md`, `vpn_options.md`
- Added customer guide and changelog

## 2026-02-27 — Unified CLI with setup detection

- Merged all scripts into single `privpn` CLI with interactive menu
- Setup detects existing WireGuard install, offers show QR / show config / reinstall
- CLI args: `privpn setup`, `privpn start`, `privpn stop`, `privpn status`, `privpn qr`

## 2026-02-27 — Pass-based key paths

- Moved VPS host, user, password to `privpn/server/` paths in `pass`
- All scripts read credentials from `pass` at runtime

## 2026-02-27 — IPv6 leak fix

- `vpn-up.sh` disables IPv6 system-wide when tunnel is up
- `vpn-down.sh` re-enables IPv6 on disconnect
- Self-healing: re-enables IPv6 first if previous run crashed
- Documented phone vs laptop IPv6 handling

## 2026-02-27 — VPS setup documentation

- Added `vps_setup.md` with kernel setup, WireGuard install, firewall config
- Documented how to add new devices

## 2026-02-27 — Laptop client and VPN comparison

- Added `vpn-up.sh` and `vpn-down.sh` for laptop
- Added `vpn_options.md` comparing WireGuard, Tailscale, OpenVPN, SSH SOCKS, Shadowsocks

## 2026-02-27 — Initial setup

- WireGuard VPN through a self-hosted VPS
- `setup.sh` for VPS install, `qr.sh` for phone QR, `status.sh` for VPS check
- All keys stored in `pass` password manager
