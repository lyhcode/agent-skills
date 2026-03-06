---
name: tailscale-cli
description: >
  Manage Tailscale VPN using the tailscale CLI.
  Use when the user wants to connect/disconnect devices, manage exit nodes,
  share files via Taildrop, expose services with Tailscale Serve or Funnel,
  check network status, manage Tailnet Lock, or configure Tailscale settings.
  Triggers on mentions of Tailscale, tailnet, MagicDNS, Taildrop, or mesh VPN.
---

# Tailscale CLI Assistant

Help users manage Tailscale VPN using the `tailscale` CLI.

## Prerequisites

- Tailscale installed on the device
- Verify setup: `tailscale status`

## CLI Path by Platform

**IMPORTANT:** The CLI path differs by platform and installation method.

### macOS (App Store)

The CLI is bundled inside the app. Use the full path or create an alias:

```bash
# Direct execution
/Applications/Tailscale.app/Contents/MacOS/Tailscale <command>

# Recommended: add alias to ~/.zshrc or ~/.bashrc
alias tailscale="/Applications/Tailscale.app/Contents/MacOS/Tailscale"
```

### macOS (Standalone variant)

Install CLI integration from the client: **Settings > CLI integration > Install Now**.
This installs a launcher to `/usr/local/bin/tailscale`.

### Linux / Windows

The `tailscale` binary is in `$PATH` by default:

```bash
tailscale <command>
```

### Force CLI mode (macOS)

If the app intercepts the command, set:

```bash
export TAILSCALE_BE_CLI=1
```

## Core Commands

### Connect & Authenticate

```bash
tailscale up                              # Connect to Tailscale
tailscale up --auth-key=<key>             # Auto-authenticate with auth key
tailscale up --force-reauth               # Force re-authentication
tailscale down                            # Disconnect
tailscale login                           # Log in (add device to tailnet)
tailscale logout                          # Log out (expire session)
tailscale switch <account>                # Switch account (fast user switching)
tailscale switch --list                   # List available accounts
```

### Status & Info

```bash
tailscale status                          # Show connected devices
tailscale status --json                   # JSON output (detailed)
tailscale status --json | jq '.Peer[] | {Name: .HostName, IP: .TailscaleIPs[0]}'
tailscale ip                              # Show this device's Tailscale IPs
tailscale ip -4                           # IPv4 only (100.x.y.z)
tailscale ip -6                           # IPv6 only
tailscale ip <hostname>                   # Get IP of another device
tailscale whois <ip>                      # Look up device/user by IP
tailscale whois --json <ip>               # JSON output
tailscale version                         # Print version
tailscale netcheck                        # Network connectivity report
```

### Settings (`tailscale set`)

Unlike `tailscale up`, `set` only changes specified preferences (no need to pass all flags):

```bash
tailscale set --hostname=my-server        # Set device hostname
tailscale set --advertise-exit-node       # Offer as exit node
tailscale set --advertise-exit-node=false # Stop offering
tailscale set --exit-node=<ip|name>       # Use exit node
tailscale set --exit-node=               # Clear exit node
tailscale set --exit-node=auto:any        # Auto-select exit node
tailscale set --exit-node-allow-lan-access # Allow LAN with exit node
tailscale set --advertise-routes=10.0.0.0/24,192.168.1.0/24  # Subnet routes
tailscale set --accept-routes             # Accept subnet routes
tailscale set --shields-up                # Block incoming connections
tailscale set --shields-up=false          # Allow incoming
tailscale set --ssh                       # Enable Tailscale SSH server
tailscale set --ssh=false                 # Disable Tailscale SSH
tailscale set --auto-update               # Enable auto-updates
tailscale set --webclient                 # Expose web interface on :5252
```

### Tailscale Serve (Expose to Tailnet)

Share local services within your tailnet:

```bash
# Reverse proxy (most common)
tailscale serve localhost:3000            # HTTPS proxy to local port
tailscale serve --http=80 localhost:3000  # HTTP proxy
tailscale serve --bg localhost:3000       # Background (persists across reboots)

# File server
tailscale serve /path/to/directory        # Serve files

# Static text
tailscale serve text:"Hello, world!"

# Self-signed backend
tailscale serve https+insecure://localhost:8443

# With custom path
tailscale serve --set-path=/api localhost:8080

# Status and reset
tailscale serve status
tailscale serve reset

# Disable specific serve
tailscale serve --https=443 off
```

### Tailscale Funnel (Expose to Internet)

Expose services from your tailnet to the public internet:

```bash
tailscale funnel localhost:3000           # HTTPS on port 443
tailscale funnel --https=8443 localhost:3000  # Allowed: 443, 8443, 10000
tailscale funnel --bg localhost:3000      # Background (persists)
tailscale funnel status
tailscale funnel reset
tailscale funnel --https=443 off          # Disable
```

### File Transfer (Taildrop)

```bash
# Send files
tailscale file cp file.txt <hostname>:
tailscale file cp *.pdf <hostname>:
tailscale file cp - <hostname>: --name=output.txt  # From stdin

# Receive files
tailscale file get <target-directory>
tailscale file get --loop ~/Downloads     # Continuously receive
tailscale file get --conflict=overwrite .  # Overwrite existing
```

### Network & DNS

```bash
tailscale ping <hostname-or-ip>           # Ping over Tailscale
tailscale ping --c=3 <hostname>           # Max 3 pings
tailscale ping --until-direct <hostname>  # Wait for direct connection
tailscale netcheck                        # Network report (UDP, DERP, latency)
tailscale netcheck --format=json          # JSON output
tailscale dns status                      # DNS forwarder & MagicDNS config
tailscale nc <hostname> <port>            # Netcat over Tailscale
tailscale ssh user@<hostname>             # SSH over Tailscale
```

### Certificates

```bash
tailscale cert hostname.ts.net                        # Generate Let's Encrypt cert
tailscale cert --cert-file=cert.pem --key-file=key.pem hostname.ts.net
tailscale cert --serve-demo hostname.ts.net            # Demo server on :443
```

Certificates have 90-day expiry (Let's Encrypt).

### Tailnet Lock

```bash
tailscale lock status                     # Check lock state
tailscale lock status --json
tailscale lock init <tlpub:key1> <tlpub:key2>  # Initialize
tailscale lock add <tlpub:key>            # Add trusted key
tailscale lock remove <tlpub:key>         # Remove trusted key
tailscale lock sign <nodekey:xxx>         # Sign a node
tailscale lock disable <disablement-secret>  # Disable lock
tailscale lock log                        # View lock changes
```

### Other Commands

```bash
tailscale update                          # Update client
tailscale update --yes                    # Update without prompt
tailscale update --dry-run                # Preview update
tailscale bugreport                       # Generate bug report
tailscale bugreport --diagnose            # With verbose diagnostics
tailscale drive share <name> <path>       # Share directory (Taildrive)
tailscale drive list                      # List shares
tailscale drive unshare <name>            # Remove share
tailscale exit-node list                  # List exit nodes
tailscale exit-node suggest               # Suggest best exit node
tailscale configure kubeconfig <host>     # Configure kubectl access
tailscale configure synology              # Configure Synology
tailscale metrics print                   # Show client metrics
tailscale completion zsh > "${fpath[1]}/_tailscale"  # Shell completion
```

## Tab Completion

```bash
# Bash (Linux)
tailscale completion bash > /etc/bash_completion.d/tailscale

# Bash (macOS with brew)
tailscale completion bash > $(brew --prefix)/etc/bash_completion.d/tailscale

# Zsh
echo "autoload -U compinit; compinit" >> ~/.zshrc
tailscale completion zsh > "${fpath[1]}/_tailscale"

# Fish
tailscale completion fish > ~/.config/fish/completions/tailscale.fish
```

## Common Patterns

### Set up as exit node

```bash
tailscale set --advertise-exit-node
# On other devices:
tailscale set --exit-node=<exit-node-name>
tailscale set --exit-node-allow-lan-access
```

### Expose a local dev server

```bash
# To tailnet only
tailscale serve --bg localhost:3000

# To the internet
tailscale funnel --bg localhost:3000
```

### Send files between devices

```bash
# On sender
tailscale file cp report.pdf my-laptop:

# On receiver
tailscale file get ~/Downloads
```

### Automate with auth key

```bash
tailscale up --auth-key=tskey-auth-xxx --hostname=ci-runner --advertise-tags=tag:ci
```

For complete command reference, see [reference/commands.md](reference/commands.md).
