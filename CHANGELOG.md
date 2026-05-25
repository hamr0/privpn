# Changelog

## 2026-05-25 — Code-review fixes

- Fix peer-IP collision: adding a peer after removing a non-last one reused an
  in-use address (it counted peers instead of finding a free octet); now scans
  AllowedIPs and assigns the lowest free IP
- Guard non-numeric "number of peers" input (was aborting under `set -e`)
- Validate the saved peer name before using it in a remote path (tampered file)
- Menu hint now reads "install / add / remove / QR"; drop a dead loop counter

## 2026-05-25 — Security hardening (audit fixes)

- SSH bootstrap no longer auto-trusts an unknown host key: the key-copy now
  shows the fingerprint and asks before the VPS password is sent, with a prompt
  to verify it against the provider console (first-connect MITM)
- Local WireGuard config is created 0600 before writing, closing a brief window
  where the private key was world-readable (tee → chmod race)
- Validate VPS host/user and peer names (`[A-Za-z0-9._:-]`): blocks shell
  metacharacters reaching the sourced config and path traversal via peer names
- `disable-password` now sets `PasswordAuthentication no` whether the directive
  is present, commented, or absent, neutralizes `sshd_config.d` drop-ins,
  validates with `sshd -t`, and fails loudly instead of reporting false success

## 2026-05-25 — README: trim Related projects

- Cut the prose and table down to one line plus clean named links
- Reframed the other repos as alternatives (not "same author")

## 2026-05-25 — v0.3.0: userspace fallback, remove-peer, Docker on the roadmap

- Backend detection on install: probes the WireGuard kernel module, falls back to
  userspace `wireguard-go` when it can't load, and fails loudly when neither works
  (so OpenVZ-style VPSes report clearly instead of dying mid-setup)
- Added **remove a peer**: drops the peer's `[Peer]` block from `wg0.conf`, deletes
  its client config, reloads WireGuard, and forgets it locally if `connect` used it
- Menu reworked: Add / Remove / Show QR / Reinstall
- Added `PRD.md` documenting principles, current state, and the planned optional
  Docker deployment mode
- README now credits prior art ([hwdsl2/wireguard-install](https://github.com/hwdsl2/wireguard-install),
  [hwdsl2/docker-wireguard](https://github.com/hwdsl2/docker-wireguard)) instead of
  reinventing the server-side installers
- Ideas borrowed: userspace fallback and peer add/remove from the hwdsl2 projects

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
