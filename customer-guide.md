# privpn Customer Guide

Complete setup walkthrough for privpn — a personal WireGuard VPN through your own VPS.

## What you need

- A VPS with a public IP address (any provider: Hetzner, DigitalOcean, Vultr, etc.)
- A laptop running Linux (Fedora, Ubuntu, Debian, etc.)
- A phone with the WireGuard app installed (optional)

## Step 1: Install privpn on your laptop

```bash
git clone https://github.com/hamr0/privpn.git
cd privpn
sudo ln -sf "$(pwd)/privpn" /usr/local/bin/privpn
```

Install WireGuard tools if you don't have them:

```bash
sudo dnf install wireguard-tools    # Fedora/RHEL
sudo apt install wireguard          # Debian/Ubuntu
```

## Step 2: Set up VPS access

Run `privpn` and choose option **1) VPS access**.

```
VPS Access Setup
Connect to your VPS and copy your SSH key.
You only need to do this once.

VPS host (IP or domain): 203.0.113.50
VPS user [root]: root
```

This will:
1. Generate an SSH key if you don't have one
2. Copy it to the VPS (enter password one last time)
3. Save the host and username to `~/.config/privpn/config`

From now on, all SSH connections use key auth — no more passwords.

## Step 3: Install WireGuard

Run `privpn` and choose option **2) Install WireGuard**.

This will:
1. Install WireGuard and dependencies on the VPS
2. Generate all keys on the server (nothing stored locally)
3. Ask how many devices to connect and their names/types
4. Show setup instructions per device (QR for phone, auto-connect for laptop)

### Laptop peer

For a Linux laptop peer, the instructions will say:

```
1. Install WireGuard (if not already):
   sudo dnf install wireguard-tools

2. Connect:
   privpn connect

   Config is fetched from the VPS automatically.
```

That's it. `privpn connect` SSHes into the VPS, downloads your config, saves it to `/etc/wireguard/privpn.conf`, and brings up the tunnel.

### Phone peer

For a phone peer, a QR code is displayed:

```
1. Install WireGuard app (App Store / Play Store)
2. Open the app > tap + > Scan from QR code
3. Scan the QR code
4. Give it a name, toggle the switch to connect
```

## Step 4: Daily use

### Connect

```bash
privpn connect     # or: privpn → 3
```

### Disconnect

```bash
privpn disconnect  # or: privpn → 4
```

### Check status

```bash
privpn status      # or: privpn → 5
```

Shows the WireGuard interface, peer info, and your current public IP.

## Managing peers

Run `privpn` → **2) Install WireGuard**. If WireGuard is already installed, you get:

```
WireGuard already running on VPS.

Current peers:
    amr-laptop
    amr-phone

1) Add new peer
2) Show peer config / QR
3) Reinstall (regenerate all keys)
0) Cancel
```

- **Add new peer** — generates new keys, adds to server config, shows instructions
- **Show peer config / QR** — retrieve an existing peer's config or re-display QR
- **Reinstall** — wipes everything, regenerates all keys (existing peers stop working)

## Architecture

```
Phone (anywhere)  ──┐
                    ├──> WireGuard tunnel (UDP 51820) ──> VPS
Laptop (anywhere) ──┘                                      └──> Internet

VPN subnet: 10.100.0.0/24
  Server:  10.100.0.1
  Peer 1:  10.100.0.2
  Peer 2:  10.100.0.3
  ...
```

### What happens when you connect

1. `privpn connect` SSHes into the VPS and reads your peer config
2. Saves it to `/etc/wireguard/privpn.conf` on your laptop
3. Disables IPv6 (prevents leaks — see below)
4. Brings up the `privpn` WireGuard interface
5. All IPv4 traffic now routes through the VPS
6. Your public IP becomes the VPS IP

### What happens when you disconnect

1. Tears down the `privpn` WireGuard interface
2. Re-enables IPv6
3. Traffic goes back to your normal connection

### Interface name

privpn uses the interface name `privpn` (not `wg0`). This means it won't conflict with other WireGuard tunnels (e.g., fedvpn from privcloud).

## Where things are stored

### On your laptop

| Location | What |
|----------|------|
| `~/.config/privpn/config` | VPS host and username |
| `~/.config/privpn/peer_name` | Which peer config to fetch |
| `/etc/wireguard/privpn.conf` | WireGuard config (fetched from VPS) |

### On the VPS

| Location | What |
|----------|------|
| `/etc/wireguard/wg0.conf` | Server config with all peers |
| `/etc/wireguard/<name>.conf` | Individual peer configs |
| `/etc/sysctl.d/99-wireguard.conf` | IP forwarding setting |

### No secrets in the repo

All keys are generated on the VPS and stay there. The repo contains only the `privpn` script and documentation.

## IPv6 leak prevention

Without protection, your real IPv6 address leaks to websites even while the VPN is active (the tunnel only routes IPv4).

**Laptop:** handled automatically. `privpn connect` disables IPv6 system-wide after the tunnel is up, `privpn disconnect` re-enables it.

**Phone:** the phone config only routes IPv4 (`AllowedIPs = 0.0.0.0/0`). Most mobile apps prefer IPv4 when available. Do **not** add `::/0` — it breaks the tunnel since most VPS have no IPv6 configured for WireGuard.

## Optional features

### Disable password login (option 6)

Disables SSH password authentication on the VPS. After this, only your SSH key can log in. This stops brute-force attacks (you may see thousands of failed attempts in `/var/log/btmp`).

**Warning:** if you lose your SSH key and have no VPS console access, you'll be locked out. Back up your key first (option 7).

### Save to pass (option 7)

If you use the `pass` password manager, this backs up everything needed to restore from scratch:

```
privpn/
├── server/
│   ├── host                    # VPS IP or domain
│   └── user                    # SSH username
├── ssh/
│   ├── private_key             # SSH private key
│   └── public_key              # SSH public key
└── wireguard/
    ├── laptop_conf             # Local WireGuard config
    ├── peer_name               # Which peer you connect as
    ├── server_conf             # Full server wg0.conf (all keys, all peers)
    └── peers/
        ├── amr-laptop          # Laptop peer config
        └── amr-phone           # Phone peer config
```

All entries are overwritten on each save (idempotent). Peer configs are fetched from the VPS.

To restore on a new machine:
```bash
# SSH key
pass show privpn/ssh/private_key > ~/.ssh/id_ed25519
chmod 600 ~/.ssh/id_ed25519

# Laptop WireGuard config
pass show privpn/wireguard/laptop_conf | sudo tee /etc/wireguard/privpn.conf > /dev/null

# Or just run privpn setup again — it fetches everything from the VPS
```

## Troubleshooting

### VPN connects but no internet

Check the VPS:
```bash
ssh root@<vps-ip>

# IP forwarding enabled?
sysctl net.ipv4.ip_forward
# Should print: net.ipv4.ip_forward = 1

# Masquerade active?
iptables -t nat -L POSTROUTING -n
# Should show MASQUERADE rule for 10.100.0.0/24

# WireGuard running?
wg show wg0
# Should show interface and peers
```

Fix if broken:
```bash
# Re-enable IP forwarding
echo 'net.ipv4.ip_forward = 1' > /etc/sysctl.d/99-wireguard.conf
sysctl -p /etc/sysctl.d/99-wireguard.conf

# Re-add masquerade
IFACE=$(ip route | grep default | awk '{print $5}' | head -1)
iptables -t nat -A POSTROUTING -s 10.100.0.0/24 -o $IFACE -j MASQUERADE

# Restart WireGuard
systemctl restart wg-quick@wg0
```

### VPN won't connect

```bash
# Is WireGuard installed on your laptop?
wg --version

# Is the VPS reachable?
ssh root@<vps-ip> "echo ok"

# Is WireGuard running on VPS?
ssh root@<vps-ip> "wg show wg0"

# Is UDP 51820 open?
ssh root@<vps-ip> "firewall-cmd --list-ports"
# Should include 51820/udp
```

### Phone shows handshake but no traffic

DNS might be blocked. Try changing DNS in the phone config to `9.9.9.9` or `208.67.222.222`.

### Websites still detect your real location

IPv6 is leaking. On laptop, verify:
```bash
cat /proc/sys/net/ipv6/conf/all/disable_ipv6
# Should print: 1 (while VPN is up)
```

If it's `0`, disconnect and reconnect:
```bash
privpn disconnect
privpn connect
```

### After VPS reboot

WireGuard auto-starts on boot (`systemctl enable wg-quick@wg0`). Your laptop just needs to reconnect:
```bash
privpn connect
```

### "Already connected" but traffic not routing

You may have another WireGuard interface up (e.g., `wg0` from another tool). Check:
```bash
sudo wg show
```

If there's a `wg0` interface that's not privpn, bring it down first:
```bash
sudo wg-quick down wg0
privpn connect
```

## Supported VPS operating systems

The install step detects the package manager and installs accordingly:

| OS | Package manager | Firewall |
|----|----------------|----------|
| AlmaLinux / RHEL / CentOS | `dnf` | `firewall-cmd` |
| Ubuntu / Debian | `apt` | `ufw` |
| Fedora | `dnf` | `firewall-cmd` |

## Security notes

- All WireGuard keys are generated on the VPS using `wg genkey` / `wg pubkey` / `wg genpsk`
- Keys never transit the network in plaintext (generated and stored on VPS)
- SSH key auth replaces password auth after initial setup
- Peer configs are stored with mode 600 on the VPS
- Local config (`/etc/wireguard/privpn.conf`) is stored with mode 600
- The `~/.config/privpn/config` file only contains the VPS hostname and username (no secrets)
- `pass` integration is optional — for users who want encrypted backups of their SSH key
