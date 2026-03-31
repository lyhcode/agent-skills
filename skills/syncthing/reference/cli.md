# Syncthing CLI Reference

## Table of Contents

- [Installation](#installation)
- [Platform Paths](#platform-paths)
- [Daemon (Serve) Command](#daemon-serve-command)
- [Utility Commands](#utility-commands)
- [CLI Subcommands](#cli-subcommands)
- [Debug Commands](#debug-commands)
- [Environment Variables](#environment-variables)
- [Network Ports](#network-ports)
- [Companion Services](#companion-services)

---

## Installation

### macOS

```bash
# Homebrew (CLI only)
brew install syncthing

# macOS GUI app (includes bundled syncthing binary)
brew install --cask syncthing-app
```

### Linux

```bash
# Debian/Ubuntu (official repo)
sudo apt install syncthing

# Or download from https://syncthing.net/downloads/
```

### Docker

```bash
docker run -d \
  -p 8384:8384 -p 22000:22000/tcp -p 22000:22000/udp -p 21027:21027/udp \
  -v /path/to/config:/var/syncthing/config \
  -v /path/to/data:/var/syncthing/data \
  syncthing/syncthing
```

---

## Platform Paths

### macOS

| Item | Path |
|------|------|
| Config directory | `~/Library/Application Support/Syncthing/` |
| Config file | `~/Library/Application Support/Syncthing/config.xml` |
| Log file | `~/Library/Application Support/Syncthing/syncthing.log` |
| Database | `~/Library/Application Support/Syncthing/index-v0.14.0.db/` |
| Certificates | `~/Library/Application Support/Syncthing/cert.pem`, `key.pem` |
| macOS app preferences | `com.github.xor-gate.syncthing-macosx` (via `defaults`) |

### Linux

| Item | Path |
|------|------|
| Config directory | `~/.local/state/syncthing/` or `~/.config/syncthing/` |
| Config file | `<config-dir>/config.xml` |

### Windows

| Item | Path |
|------|------|
| Config directory | `%LOCALAPPDATA%\Syncthing\` |

Override paths with `--config`, `--data`, or `--home` flags, or `STCONFDIR` / `STDATADIR` / `STHOMEDIR` environment variables.

---

## Daemon (Serve) Command

Start the Syncthing daemon:

```bash
syncthing serve [flags]
# or simply:
syncthing [flags]
```

### Key Flags

| Flag | Description |
|------|-------------|
| `--config=PATH` | Configuration directory |
| `--data=PATH` | Data directory (database, logs) |
| `--home=PATH` | Set both config and data directory |
| `--gui-address=ADDR` | Override GUI listen address (e.g., `http://0.0.0.0:8384`) |
| `--gui-apikey=KEY` | Override API key |
| `--no-browser` | Don't open browser on startup |
| `--no-restart` | Disable automatic restarts |
| `--no-upgrade` | Disable automatic upgrades |
| `--paused` | Start with all devices/folders paused |
| `--logfile=PATH` | Write logs to file (use `-` for stdout) |
| `--log-level=LEVEL` | Log level: `DEBUG`, `INFO`, `WARN`, `ERROR` |
| `--log-max-size=SIZE` | Max log file size (default: 10 MiB) |
| `--log-max-old-files=N` | Rotated log files to keep (default: 3) |
| `--audit` | Write events to audit file |
| `--auditfile=PATH` | Audit file path (`-` for stdout, `--` for stderr) |
| `--allow-newer-config` | Allow loading config from newer Syncthing version |
| `--no-port-probing` | Don't probe for available GUI port |

### Database Flags

| Flag | Default | Description |
|------|---------|-------------|
| `--db-maintenance-interval` | `8h` | Database maintenance frequency |
| `--db-delete-retention-interval` | `10920h` | Retention time for deleted items |

### Debug Flags

| Flag | Description |
|------|-------------|
| `--debug-gui-assets=PATH` | Custom GUI assets directory |
| `--debug-perf-stats=PATH` | Write performance stats to CSV |
| `--debug-profile-cpu` | Write CPU profile on exit |
| `--debug-profile-heap` | Write heap profiles on memory increase |
| `--debug-profile-block` | Write block profiles every 20s |
| `--debug-profiler-listen=ADDR` | Network profiler address |

---

## Utility Commands

```bash
# Show version
syncthing version

# Show device ID
syncthing device-id

# Show configuration paths
syncthing paths

# Generate keys and initial config (without starting)
syncthing generate --config=PATH

# Open web GUI in browser
syncthing browser

# Check for upgrades (--check-only to just check without installing)
syncthing upgrade [--check-only]

# Decrypt files from an untrusted/encrypted device
syncthing decrypt --to=OUTPUT_DIR --password=PASS ENCRYPTED_DIR
```

---

## CLI Subcommands

Runtime control via `syncthing cli`:

```bash
syncthing cli [--gui-address=ADDR] [--gui-apikey=KEY] <subcommand>
```

### Config Management

```bash
# Get full config as JSON
syncthing cli config dump-json

# Get/set specific config sections
syncthing cli config devices list
syncthing cli config folders list
syncthing cli config options get
```

### Operations

```bash
# Show system status
syncthing cli show system

# Show connections
syncthing cli show connections

# Show pending devices
syncthing cli show pending-devices

# Show pending folders
syncthing cli show pending-folders

# Show discovery
syncthing cli show discovery
```

### Error Management

```bash
# List errors
syncthing cli errors show

# Clear errors
syncthing cli errors clear
```

---

## Debug Commands

```bash
# Database statistics
syncthing debug database-statistics

# Database file counts
syncthing debug database-counts FOLDER_ID

# File metadata from database
syncthing debug database-file FOLDER_ID FILE_NAME

# Reset database
syncthing debug reset-database
```

---

## Environment Variables

### Logging & Tracing

| Variable | Description |
|----------|-------------|
| `STTRACE` | Enable debug tracing. Comma-separated facility list. |
| `STDEADLOCKTIMEOUT` | Deadlock detection timeout |

**STTRACE facilities:** `api`, `beacon`, `config`, `connections`, `db/sqlite`, `dialer`, `discover`, `events`, `fs`, `model`, `nat`, `protocol`, `relay/client`, `scanner`, `stun`, `upgrade`, `upnp`

Set specific levels: `STTRACE=facility:LEVEL` (e.g., `STTRACE=api:5`)

### Path Overrides

| Variable | Description |
|----------|-------------|
| `STCONFDIR` | Configuration directory |
| `STDATADIR` | Data directory |
| `STHOMEDIR` | Both config and data directory |

### Performance Tuning

| Variable | Default | Description |
|----------|---------|-------------|
| `GOMAXPROCS` | (all CPUs) | Limit CPU cores used |
| `GOGC` | `100` | GC sensitivity (lower = more frequent GC) |
| `STVERSIONEXTRA` | — | Custom version string suffix |

---

## Network Ports

| Port | Protocol | Purpose |
|------|----------|---------|
| 8384/TCP | HTTP(S) | Web GUI and REST API |
| 22000/TCP | BEP over TLS | Sync protocol (TCP) |
| 22000/UDP | BEP over QUIC | Sync protocol (QUIC) |
| 21027/UDP | — | Local discovery broadcasts |

### Firewall Configuration

For direct connections (bypassing relay), open:
- **Inbound:** 22000/TCP and 22000/UDP
- **Local discovery:** 21027/UDP (LAN only)
- **GUI access (if remote):** 8384/TCP

---

## Companion Services

### Discovery Server (`stdiscosrv`)

Helps devices find each other across the internet.

```bash
stdiscosrv [flags]
```

### Relay Server (`strelaysrv`)

Relays traffic between devices that can't connect directly (NAT/firewall).

```bash
strelaysrv [flags]
```

Relay connections are end-to-end encrypted but significantly slower than direct connections.

---

## macOS App Management

The macOS GUI app (`syncthing-macos`) can be configured via `defaults`:

```bash
# View all settings
defaults read com.github.xor-gate.syncthing-macosx

# Auto-start on login
defaults write com.github.xor-gate.syncthing-macosx StartAtLogin 1

# Disable automatic update checks
defaults write com.github.xor-gate.syncthing-macosx SUEnableAutomaticChecks 0

# Use a custom syncthing binary
defaults write com.github.xor-gate.syncthing-macosx Executable /usr/local/bin/syncthing

# Pass extra CLI arguments (no spaces)
defaults write com.github.xor-gate.syncthing-macosx Arguments '--audit --no-browser'

# Reset to bundled binary
defaults delete com.github.xor-gate.syncthing-macosx Executable
```

### Auto-Start (without macOS app)

**Homebrew launchd service:**
```bash
brew services start syncthing
```

**Manual launchd plist:**
Place in `~/Library/LaunchAgents/com.syncthing.syncthing.plist`

### Linux Systemd

```bash
# User service
systemctl --user enable syncthing
systemctl --user start syncthing

# System service (for a specific user)
sudo systemctl enable syncthing@username
sudo systemctl start syncthing@username
```
