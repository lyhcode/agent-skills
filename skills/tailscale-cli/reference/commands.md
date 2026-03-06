# Tailscale CLI Command Reference

Full command reference for `tailscale` CLI. Source: https://tailscale.com/docs/reference/tailscale-cli

## CLI Path by Platform

| Platform | Path | Notes |
|----------|------|-------|
| Linux | `tailscale` | In `$PATH` by default |
| macOS (App Store) | `/Applications/Tailscale.app/Contents/MacOS/Tailscale` | Create alias recommended |
| macOS (Standalone) | `/usr/local/bin/tailscale` | After CLI integration install |
| Windows | `tailscale` | `.exe` in `$PATH` |
| iOS / Android | N/A | No CLI support |

### macOS Setup

```bash
# App Store version: add alias to shell config
alias tailscale="/Applications/Tailscale.app/Contents/MacOS/Tailscale"

# Standalone: install CLI integration
# Settings > CLI integration > Show me how > Install Now
# Installs to /usr/local/bin/tailscale

# Force CLI mode (if app intercepts)
export TAILSCALE_BE_CLI=1
```

## Tab Completion

```bash
tailscale completion <shell> [--flags=<bool>] [--descs=<bool>]
```

| Shell | Setup |
|-------|-------|
| Bash (Linux) | `tailscale completion bash > /etc/bash_completion.d/tailscale` |
| Bash (macOS) | `tailscale completion bash > $(brew --prefix)/etc/bash_completion.d/tailscale` |
| Zsh | `tailscale completion zsh > "${fpath[1]}/_tailscale"` |
| Fish | `tailscale completion fish > ~/.config/fish/completions/tailscale.fish` |
| PowerShell | `tailscale completion powershell \| Out-String \| Invoke-Expression` |

---

## tailscale up

Connect to Tailscale and authenticate.

```
tailscale up [flags]
```

**Flags are NOT persisted between runs.** You must specify all flags each time. In v1.8+, the CLI warns if you forget previously-used flags.

| Flag | Description |
|------|-------------|
| `--accept-dns` | Accept DNS configuration from admin console |
| `--accept-risk=<risk>` | Skip confirmation (`lose-ssh`, `all`, or empty) |
| `--accept-routes` | Accept subnet routes (default varies by OS) |
| `--advertise-connector` | Offer as app connector |
| `--advertise-exit-node` | Offer as exit node |
| `--advertise-routes=<ip>` | Expose subnet routes (comma-separated CIDRs) |
| `--advertise-tags=<tags>` | Apply tagged permissions (must be in TagOwners) |
| `--auth-key=<key>` | Auto-authenticate with auth key |
| `--client-id` | Client ID for workload identity federation |
| `--client-secret` | OAuth secret (prefix `file:` for file path) |
| `--exit-node=<ip\|name>` | Use exit node (`auto:any` for auto, empty to clear) |
| `--exit-node-allow-lan-access` | Allow LAN while using exit node |
| `--force-reauth` | Force re-authentication |
| `--hostname=<name>` | Set device hostname (changes MagicDNS name) |
| `--id-token` | Identity provider token (prefix `file:` for file) |
| `--audience` | Audience for ID token |
| `--json` | JSON output |
| `--login-server=<url>` | Control server URL (default: controlplane.tailscale.com) |
| `--netfilter-mode=<mode>` | Linux only: `on` (default), `nodivert`, `off` |
| `--operator=<user>` | Unix user to operate tailscaled |
| `--qr` | Generate QR code for login URL |
| `--qr-format=<format>` | QR size: `small` (default), `large` |
| `--reset` | Reset unspecified settings to defaults |
| `--shields-up` | Block incoming connections |
| `--snat-subnet-routes` | Source NAT advertised routes (Linux, default: true) |
| `--ssh` | Run Tailscale SSH server |
| `--stateful-filtering` | Enable stateful filtering (Linux) |
| `--timeout=<duration>` | Max wait for service init (default: `0s` = forever) |
| `--unattended` | Windows only: keep running after user logout |

### accept-routes defaults by OS

| OS | Default |
|----|---------|
| Windows, iOS, Android, macOS | Accept (yes) |
| Linux, other | Do not accept (no) |

---

## tailscale down

Disconnect from Tailscale.

```
tailscale down [flags]
```

| Flag | Description |
|------|-------------|
| `--accept-risk=<risk>` | Skip confirmation (`lose-ssh`, `all`) |
| `--reason="<desc>"` | Disconnection reason (with AlwaysOn policies) |

---

## tailscale login

Log in and add device to tailnet. Same flags as `tailscale up` plus:

```
tailscale login [flags]
```

| Additional Flag | Description |
|-----------------|-------------|
| `--nickname=<name>` | Nickname for account |

---

## tailscale logout

Log out and expire session.

```
tailscale logout [--reason="<desc>"]
```

Ephemeral nodes are removed immediately on logout.

---

## tailscale set

Change specific preferences without resetting others (unlike `tailscale up`).

```
tailscale set [flags]
```

| Flag | Description |
|------|-------------|
| `--accept-dns` | Accept DNS configuration |
| `--accept-risk=<risk>` | Skip confirmation |
| `--accept-routes` / `=false` | Accept/reject subnet routes |
| `--advertise-connector` | Offer as app connector |
| `--advertise-exit-node` / `=false` | Offer/stop offering as exit node |
| `--advertise-routes=<cidrs>` | Expose subnet routes |
| `--auto-update` / `=false` | Enable/disable auto-updates |
| `--exit-node=<ip\|name>` | Use exit node (`auto:any` for suggested) |
| `--exit-node-allow-lan-access` / `=false` | Allow LAN with exit node |
| `--hostname=<name>` | Set device hostname |
| `--netfilter-mode=<mode>` | Netfilter: `on`/`nodivert`/`off` |
| `--nickname=<name>` | Account nickname |
| `--operator=<user>` | Unix user for tailscaled |
| `--relay-server-port=<port>` | UDP port for peer relay |
| `--report-posture` | Allow device posture reporting |
| `--shields-up` / `=false` | Block/allow incoming connections |
| `--snat-subnet-routes` | Source NAT for routes |
| `--ssh` / `=false` | Enable/disable Tailscale SSH |
| `--stateful-filtering` | Stateful filtering |
| `--update-check` | Notify about updates |
| `--webclient` / `=false` | Expose web interface on `:5252` |

---

## tailscale status

Show connection status.

```
tailscale status [flags]
```

| Flag | Description |
|------|-------------|
| `--active` | Show only active peers |
| `--json` | Detailed JSON output |
| `--peers` | Show peers (default: true) |
| `--self` | Show local machine (default: true) |
| `--header` | Show column headers (default: false) |
| `--web` | Run HTML status webserver |
| `--listen=<addr>` | Web listen address (default: 127.0.0.1:8384) |

Output columns: Tailscale IP | machine name | owner | OS | connection status

Connection status: `active` (traffic), `idle` (no traffic), `-` (never connected)

```bash
# Example: count peers by relay server
tailscale status --json | jq -r '.Peer[].Relay | select(.!="")' | sort | uniq -c | sort -nr
```

---

## tailscale ip

Get Tailscale IP addresses.

```
tailscale ip [flags] [<hostname>]
```

| Flag | Description |
|------|-------------|
| `--4` | IPv4 only (100.x.y.z) |
| `--6` | IPv6 only |
| `--1` | One address, prefer IPv4 |

---

## tailscale ping

Ping another device over Tailscale.

```
tailscale ping [flags] <hostname-or-ip>
```

| Flag | Description |
|------|-------------|
| `--c=<n>` | Max pings (default: 10) |
| `--icmp` | ICMP-level ping |
| `--tsmp` | TSMP-level ping |
| `--peerapi` | Hit PeerAPI HTTP server |
| `--size=<bytes>` | Ping message size (0 = minimum) |
| `--timeout=<duration>` | Max wait (default: 5s) |
| `--until-direct` | Stop when direct path established (default: true) |
| `--verbose` | Verbose output |

---

## tailscale netcheck

Network connectivity report.

```
tailscale netcheck [flags]
```

| Flag | Description |
|------|-------------|
| `--every=<duration>` | Repeat frequency |
| `--format=<fmt>` | Output: empty, `json`, `json-line` |
| `--verbose` | Verbose logs |

Reports: UDP status, IPv4/IPv6 support, NAT type, DERP latency.

---

## tailscale serve

Share local services within your tailnet.

```
tailscale serve [flags] <target>
tailscale serve <subcommand>
```

### Subcommands

| Subcommand | Description |
|------------|-------------|
| `status` | Current status (`--json` for JSON) |
| `reset` | Clear all serve configuration |
| `drain <service>` | Gracefully remove service host |
| `advertise <service>` | Register as service host |
| `get-config <file>` | Export service config |
| `set-config <file>` | Apply service config |

### Flags

| Flag | Description |
|------|-------------|
| `--bg` | Background process (persists across reboots) |
| `--http=<port>` | HTTP server |
| `--https=<port>` | HTTPS server (default) |
| `--tcp=<port>` | Raw TCP forwarder |
| `--tls-terminated-tcp=<port>` | TLS-terminated TCP |
| `--proxy-protocol=<1\|2>` | PROXY protocol version |
| `--set-path=<path>` | Append path to URL |
| `--service=<vip>` | Serve for service with virtual IP |
| `--tun` | Layer 3 service forwarding |
| `--yes` | Skip prompts |
| `--accept-app-caps=<caps>` | Forward app capabilities |

### Target Types

```bash
# Reverse proxy
tailscale serve localhost:3000
tailscale serve --http=80 localhost:3000
tailscale serve https+insecure://localhost:8443   # Self-signed backend

# File server (absolute paths only)
tailscale serve /home/alice/blog/

# Static text
tailscale serve text:"Hello, world!"

# Disable
tailscale serve --https=443 off
```

---

## tailscale funnel

Expose services to the public internet.

```
tailscale funnel [flags] <target>
```

Same flags and target types as `serve`, with restrictions:
- Allowed HTTPS ports: **443, 8443, 10000**

### Subcommands

| Subcommand | Description |
|------------|-------------|
| `status` | Current status (`--json`) |
| `reset` | Clear funnel config |

```bash
tailscale funnel localhost:3000
tailscale funnel --https=8443 localhost:3000
tailscale funnel --bg localhost:3000       # Persists
tailscale funnel --https=443 off           # Disable
```

---

## tailscale file (Taildrop)

Transfer files between devices.

### file cp

```
tailscale file cp [flags] <files...> <target>:
```

| Flag | Description |
|------|-------------|
| `--name=<name>` | Alternate filename (useful with stdin `-`) |
| `--targets` | List possible targets |
| `--verbose` | Verbose output |

### file get

```
tailscale file get [flags] <target-directory>
```

| Flag | Description |
|------|-------------|
| `--conflict=<mode>` | `skip`, `overwrite`, or `rename` |
| `--loop` | Continuously receive files |
| `--verbose` | Verbose output |
| `--wait` | Wait if inbox empty |

---

## tailscale cert

Generate Let's Encrypt TLS certificates (90-day expiry).

```
tailscale cert [flags] <hostname.ts.net>
```

| Flag | Description |
|------|-------------|
| `--cert-file=<path>` | Certificate output path |
| `--key-file=<path>` | Private key output path |
| `--min-validity=<duration>` | Minimum remaining validity |
| `--serve-demo` | Serve demo on `:443` |

---

## tailscale dns

Access DNS settings (v1.74.0+).

```
tailscale dns <subcommand>
```

| Subcommand | Description |
|------------|-------------|
| `status` | Print DNS forwarder and MagicDNS config (`--all` for debug) |
| `query` | DNS query via local forwarder (v1.76.0+) |

---

## tailscale lock (Tailnet Lock)

```
tailscale lock <subcommand> [flags]
```

| Subcommand | Description |
|------------|-------------|
| `init <tlpub:keys...>` | Initialize Tailnet Lock |
| `status` | Lock state (`--json`) |
| `add <tlpub:keys...>` | Add trusted signing keys |
| `remove <tlpub:keys...>` | Remove trusted keys (`--re-sign` default: true) |
| `sign <nodekey:xxx>` | Sign a node key |
| `sign <auth-key>` | Sign a pre-approved auth key |
| `disable <secret>` | Disable lock with disablement secret |
| `disablement-kdf <hex>` | Compute disablement value |
| `log` | List lock changes (`--limit`, `--json`) |
| `local-disable` | Disable lock for current node only |
| `revoke-keys <tlpub:keys...>` | Revoke lock keys (multi-step) |

### init flags

| Flag | Description |
|------|-------------|
| `--confirm=false` | Skip confirmation |
| `--gen-disablement-for-support` | Generate secret for Tailscale support |
| `--gen-disablements=<n>` | Number of disablement secrets (min 1) |

---

## tailscale switch

Switch Tailscale accounts (fast user switching).

```
tailscale switch <account> [flags]
tailscale switch remove <id>       # Remove account (alpha)
```

| Flag | Description |
|------|-------------|
| `--list` | List available accounts |

---

## tailscale update

Update Tailscale client.

```
tailscale update [flags]
```

| Flag | Description |
|------|-------------|
| `--dry-run` | Preview without updating |
| `--track=<channel>` | `stable` or `unstable` |
| `--version=<ver>` | Specific version (cannot use with --track) |
| `--yes` | Skip prompt |

---

## tailscale whois

Look up device/user by Tailscale IP.

```
tailscale whois [flags] <ip[:port]>
```

| Flag | Description |
|------|-------------|
| `--json` | JSON output |
| `--proto=<proto>` | Protocol filter: `tcp`, `udp`, or empty |

---

## tailscale exit-node

```
tailscale exit-node <subcommand>
```

| Subcommand | Description |
|------------|-------------|
| `list` | List exit nodes (`--filter=<country>`) |
| `suggest` | Recommend best exit node |

---

## tailscale drive (Taildrive)

Share directories within tailnet.

```bash
tailscale drive share <name> <path>     # Create/modify share
tailscale drive rename <old> <new>      # Rename share
tailscale drive unshare <name>          # Remove share
tailscale drive list                    # List shares
```

---

## tailscale configure

Configure external resources for tailnet.

| Subcommand | Description |
|------------|-------------|
| `kubeconfig <host>` | Configure kubectl access (`--http` for HTTP) |
| `mac-vpn install\|uninstall` | macOS VPN configuration |
| `synology` | Configure Synology |
| `sysext activate\|deactivate\|status` | macOS system extension |
| `systray` | Linux system tray app (beta, v1.88+) |

---

## Other Commands

| Command | Description |
|---------|-------------|
| `tailscale bugreport` | Generate bug report (`--diagnose` for verbose) |
| `tailscale nc <host> <port>` | Netcat over Tailscale |
| `tailscale ssh [user@]<host>` | SSH over Tailscale |
| `tailscale version` | Print version (`--daemon`, `--json`, `--upstream`) |
| `tailscale web` | Web UI for tailscaled (`--listen`, `--readonly`) |
| `tailscale licenses` | Open source license info |
| `tailscale metrics print\|write` | Client metrics |
| `tailscale syspolicy list\|reload` | System policies (`--json`) |
| `tailscale appc-routes` | App connector route status (`--all`, `--map`) |

## Global Flag

| Flag | Description |
|------|-------------|
| `--socket=<path>` | Path to tailscaled socket |
