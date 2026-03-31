---
name: syncthing
description: >
  Manage Syncthing file synchronization via the REST API and CLI.
  Use when the user wants to add/remove devices or folders, check sync status,
  configure versioning or ignore patterns, monitor connections, manage pending
  devices, pause/resume sync, troubleshoot sync issues, or automate Syncthing
  administration. Triggers on mentions of Syncthing, file sync, peer-to-peer sync,
  device pairing, .stignore, block exchange protocol, or syncthing REST API.
  Even if the user just says "check sync status" or "add a device" in a Syncthing
  context, this skill should be used.
version_target: "Syncthing 2.0"
references:
  - https://docs.syncthing.net/
  - https://docs.syncthing.net/dev/rest.html
---

# Syncthing Administration

Help users manage Syncthing file synchronization via the REST API and CLI.

## Prerequisites

- Syncthing running on the target machine
- API key available (from config.xml or the web GUI)
- Default API address: `http://127.0.0.1:8384`

### macOS Paths

The macOS app (`syncthing-macos`) stores data at:

```
~/Library/Application Support/Syncthing/
├── config.xml          # Main configuration
├── syncthing.log       # Log file
├── cert.pem / key.pem  # TLS certificates
└── index-v0.14.0.db/   # Database
```

Preferences bundle ID: `com.github.xor-gate.syncthing-macosx`

```bash
# View macOS app preferences
defaults read com.github.xor-gate.syncthing-macosx

# Auto-start on login
defaults write com.github.xor-gate.syncthing-macosx StartAtLogin 1
```

### Homebrew Installation

```bash
brew install syncthing
# Or just the macOS GUI wrapper:
brew install --cask syncthing-app
```

## Authentication

All REST API calls (except `/rest/noauth/*`) require authentication:

```bash
# API key header (preferred)
curl -H "X-API-Key: YOUR_API_KEY" http://127.0.0.1:8384/rest/system/status

# Bearer token
curl -H "Authorization: Bearer YOUR_API_KEY" http://127.0.0.1:8384/rest/system/status
```

The API key is in `config.xml` at `configuration/gui/apikey`, or visible in the web GUI under Settings > API Key.

## Common Operations

### Check System Status

```bash
# System status (device ID, uptime, memory usage)
curl -s -H "X-API-Key: $KEY" http://127.0.0.1:8384/rest/system/status | jq .

# Version info
curl -s -H "X-API-Key: $KEY" http://127.0.0.1:8384/rest/system/version | jq .

# Health check (no auth needed)
curl -s http://127.0.0.1:8384/rest/noauth/health

# Connection status for all devices
curl -s -H "X-API-Key: $KEY" http://127.0.0.1:8384/rest/system/connections | jq .
```

### Device Management

```bash
# List all configured devices
curl -s -H "X-API-Key: $KEY" http://127.0.0.1:8384/rest/config/devices | jq .

# Add a new device
curl -s -X POST -H "X-API-Key: $KEY" -H "Content-Type: application/json" \
  http://127.0.0.1:8384/rest/config/devices \
  -d '{
    "deviceID": "DEVICE-ID-HERE",
    "name": "My Phone",
    "addresses": ["dynamic"],
    "compression": "metadata",
    "autoAcceptFolders": false
  }'

# Get a specific device
curl -s -H "X-API-Key: $KEY" http://127.0.0.1:8384/rest/config/devices/DEVICE-ID | jq .

# Update device (partial update)
curl -s -X PATCH -H "X-API-Key: $KEY" -H "Content-Type: application/json" \
  http://127.0.0.1:8384/rest/config/devices/DEVICE-ID \
  -d '{"paused": true}'

# Delete a device
curl -s -X DELETE -H "X-API-Key: $KEY" http://127.0.0.1:8384/rest/config/devices/DEVICE-ID

# View pending devices (awaiting acceptance)
curl -s -H "X-API-Key: $KEY" http://127.0.0.1:8384/rest/cluster/pending/devices | jq .

# Dismiss a pending device
curl -s -X DELETE -H "X-API-Key: $KEY" \
  "http://127.0.0.1:8384/rest/cluster/pending/devices?device=DEVICE-ID"
```

### Folder Management

```bash
# List all folders
curl -s -H "X-API-Key: $KEY" http://127.0.0.1:8384/rest/config/folders | jq .

# Add a new folder shared with a device
curl -s -X POST -H "X-API-Key: $KEY" -H "Content-Type: application/json" \
  http://127.0.0.1:8384/rest/config/folders \
  -d '{
    "id": "my-documents",
    "label": "My Documents",
    "path": "/Users/kyle/Documents/Sync",
    "type": "sendreceive",
    "devices": [
      {"deviceID": "LOCAL-DEVICE-ID"},
      {"deviceID": "REMOTE-DEVICE-ID"}
    ],
    "rescanIntervalS": 3600,
    "fsWatcherEnabled": true
  }'

# Update folder (partial)
curl -s -X PATCH -H "X-API-Key: $KEY" -H "Content-Type: application/json" \
  http://127.0.0.1:8384/rest/config/folders/my-documents \
  -d '{"paused": false}'

# Delete a folder
curl -s -X DELETE -H "X-API-Key: $KEY" http://127.0.0.1:8384/rest/config/folders/my-documents

# View pending folders
curl -s -H "X-API-Key: $KEY" \
  "http://127.0.0.1:8384/rest/cluster/pending/folders?device=DEVICE-ID" | jq .
```

**Folder types:**
- `sendreceive` — Bidirectional sync (default)
- `sendonly` — Push local changes only, ignore remote
- `receiveonly` — Accept remote changes only, ignore local
- `receiveencrypted` — Receive encrypted data (for untrusted devices)

### Sync Status & Progress

```bash
# Folder sync status
curl -s -H "X-API-Key: $KEY" \
  "http://127.0.0.1:8384/rest/db/status?folder=my-documents" | jq .

# Sync completion percentage (overall or per device)
curl -s -H "X-API-Key: $KEY" \
  "http://127.0.0.1:8384/rest/db/completion?folder=my-documents" | jq .

# Per-device completion
curl -s -H "X-API-Key: $KEY" \
  "http://127.0.0.1:8384/rest/db/completion?folder=my-documents&device=DEVICE-ID" | jq .

# Files that need syncing
curl -s -H "X-API-Key: $KEY" \
  "http://127.0.0.1:8384/rest/db/need?folder=my-documents" | jq .

# Folder errors
curl -s -H "X-API-Key: $KEY" \
  "http://127.0.0.1:8384/rest/folder/errors?folder=my-documents" | jq .
```

### Pause / Resume

```bash
# Pause all devices
curl -s -X POST -H "X-API-Key: $KEY" http://127.0.0.1:8384/rest/system/pause

# Resume all devices
curl -s -X POST -H "X-API-Key: $KEY" http://127.0.0.1:8384/rest/system/resume

# Pause a specific device
curl -s -X POST -H "X-API-Key: $KEY" \
  "http://127.0.0.1:8384/rest/system/pause?device=DEVICE-ID"

# Pause a specific folder (via config patch)
curl -s -X PATCH -H "X-API-Key: $KEY" -H "Content-Type: application/json" \
  http://127.0.0.1:8384/rest/config/folders/my-documents \
  -d '{"paused": true}'
```

### Ignore Patterns

```bash
# Get ignore patterns for a folder
curl -s -H "X-API-Key: $KEY" \
  "http://127.0.0.1:8384/rest/db/ignores?folder=my-documents" | jq .

# Set ignore patterns
curl -s -X POST -H "X-API-Key: $KEY" -H "Content-Type: application/json" \
  "http://127.0.0.1:8384/rest/db/ignores?folder=my-documents" \
  -d '{"ignore": ["*.tmp", ".DS_Store", "node_modules", "(?i)thumbs.db"]}'
```

Ignore pattern syntax:
- `*.tmp` — Glob matching
- `!important.tmp` — Negation (un-ignore)
- `(?i)pattern` — Case-insensitive
- `/pattern` — Current directory only
- `**/pattern` — Any subdirectory

### Scan & Override

```bash
# Trigger a rescan of a folder
curl -s -X POST -H "X-API-Key: $KEY" \
  "http://127.0.0.1:8384/rest/db/scan?folder=my-documents"

# Rescan a specific subdirectory
curl -s -X POST -H "X-API-Key: $KEY" \
  "http://127.0.0.1:8384/rest/db/scan?folder=my-documents&sub=subfolder/path"

# Override remote changes (send-only folders)
curl -s -X POST -H "X-API-Key: $KEY" \
  "http://127.0.0.1:8384/rest/db/override?folder=my-documents"

# Revert local changes (receive-only folders)
curl -s -X POST -H "X-API-Key: $KEY" \
  "http://127.0.0.1:8384/rest/db/revert?folder=my-documents"
```

### Logs & Errors

```bash
# View system logs
curl -s -H "X-API-Key: $KEY" http://127.0.0.1:8384/rest/system/log | jq .

# View system errors
curl -s -H "X-API-Key: $KEY" http://127.0.0.1:8384/rest/system/error | jq .

# Clear errors
curl -s -X POST -H "X-API-Key: $KEY" http://127.0.0.1:8384/rest/system/error/clear
```

### Restart & Shutdown

```bash
# Restart Syncthing
curl -s -X POST -H "X-API-Key: $KEY" http://127.0.0.1:8384/rest/system/restart

# Shutdown Syncthing
curl -s -X POST -H "X-API-Key: $KEY" http://127.0.0.1:8384/rest/system/shutdown

# Check if restart is needed after config changes
curl -s -H "X-API-Key: $KEY" http://127.0.0.1:8384/rest/config/restart-required | jq .
```

## Versioning Configuration

Configure file versioning when adding or updating a folder. Four strategies are available:

**Trash Can** — Moves old files to `.stversions`, single copy per file:
```json
{"type": "trashcan", "params": {"cleanoutDays": "30"}}
```

**Simple** — Keeps N timestamped copies:
```json
{"type": "simple", "params": {"keep": "5", "cleanoutDays": "0"}}
```

**Staggered** — Tiered retention (hourly/daily/weekly buckets):
```json
{"type": "staggered", "params": {"maxAge": "365"}}
```

**External** — Delegates to a custom script:
```json
{"type": "external", "params": {"command": "/path/to/script %FOLDER_PATH% %FILE_PATH%"}}
```

Apply versioning via folder config PATCH:
```bash
curl -s -X PATCH -H "X-API-Key: $KEY" -H "Content-Type: application/json" \
  http://127.0.0.1:8384/rest/config/folders/my-documents \
  -d '{"versioning": {"type": "staggered", "params": {"maxAge": "365"}}}'
```

## Event Streaming

Subscribe to real-time events via long-polling:

```bash
# Get all events (long-poll, blocks until events arrive)
curl -s -H "X-API-Key: $KEY" "http://127.0.0.1:8384/rest/events?since=0&limit=10" | jq .

# Only disk events
curl -s -H "X-API-Key: $KEY" "http://127.0.0.1:8384/rest/events/disk?since=0" | jq .

# Filter by event type
curl -s -H "X-API-Key: $KEY" \
  "http://127.0.0.1:8384/rest/events?events=DeviceConnected,DeviceDisconnected&since=0" | jq .
```

Key event types: `DeviceConnected`, `DeviceDisconnected`, `FolderCompletion`, `FolderErrors`, `ItemFinished`, `StateChanged`, `LocalChangeDetected`, `RemoteChangeDetected`, `ConfigSaved`, `StartupComplete`.

## Troubleshooting

### Connection Issues
1. Check if Syncthing is running: `curl http://127.0.0.1:8384/rest/noauth/health`
2. Verify connections: `GET /rest/system/connections` — look for `connected: true`
3. If using relay (slow): check NAT/firewall, configure port forwarding for port 22000
4. Pending devices not appearing: both sides must add each other's device ID

### Sync Not Progressing
1. Check folder status: `GET /rest/db/status?folder=ID` — look at `state` field
2. Check for errors: `GET /rest/folder/errors?folder=ID`
3. Trigger manual rescan: `POST /rest/db/scan?folder=ID`
4. Check needed files: `GET /rest/db/need?folder=ID`

### Performance
- Relay connections are much slower than direct — check connection type in `/rest/system/connections`
- Reduce scan frequency with `rescanIntervalS` for large folders (use filesystem watcher instead)
- Set bandwidth limits via options: `GET/PATCH /rest/config/options` — `maxSendKbps`, `maxRecvKbps`

## Reference Documentation

For detailed endpoint lists and configuration options, read these reference files:

- **`reference/rest-api.md`** — Complete REST API endpoint reference with all parameters
- **`reference/config.md`** — Configuration structure: folders, devices, GUI, options, versioning, ignore patterns
- **`reference/cli.md`** — CLI commands, flags, environment variables, and macOS-specific details
