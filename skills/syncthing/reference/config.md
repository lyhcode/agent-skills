# Syncthing Configuration Reference

Configuration file: `config.xml` (location depends on platform — see CLI reference).
All settings are also accessible via the REST API at `/rest/config`.

## Table of Contents

- [Folder Configuration](#folder-configuration)
- [Device Configuration](#device-configuration)
- [GUI Configuration](#gui-configuration)
- [Options (Global Settings)](#options-global-settings)
- [LDAP Configuration](#ldap-configuration)
- [Versioning](#versioning)
- [Ignore Patterns](#ignore-patterns)
- [Defaults](#defaults)

---

## Folder Configuration

Retrieved via `GET /rest/config/folders/{id}`.

### Core Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `id` | string | — | Unique folder identifier (required) |
| `label` | string | — | Human-readable name |
| `path` | string | — | Local directory path (required) |
| `type` | string | `sendreceive` | Sync mode (see below) |
| `devices` | array | — | Devices sharing this folder (`[{"deviceID": "..."}]`) |
| `paused` | bool | `false` | Pause synchronization |
| `rescanIntervalS` | int | `3600` | Full rescan interval in seconds |
| `fsWatcherEnabled` | bool | `true` | Real-time filesystem change detection |
| `fsWatcherDelayS` | int | `10` | Delay before processing watched changes |
| `markerName` | string | `.stfolder` | Folder marker directory name |

### Folder Types

| Type | Description |
|------|-------------|
| `sendreceive` | Bidirectional sync (default) |
| `sendonly` | Only push local changes; ignore remote changes. Use "Override Changes" to force local state. |
| `receiveonly` | Only accept remote changes; ignore local edits. Use "Revert Local Changes" to restore. |
| `receiveencrypted` | Receive encrypted data for untrusted devices |

### Performance Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `maxConcurrentWrites` | int | `16` | Concurrent write operations (max 256) |
| `copiers` | int | `0` | Concurrent file copiers (0 = auto) |
| `hashers` | int | `0` | Concurrent hash workers (0 = auto) |
| `pullers` | int | `0` | Concurrent pull operations (0 = auto) |
| `pullerMaxPendingKiB` | int | `0` | Max pending data during pulls (0 = auto) |
| `pullerDelayS` | int | `1` | Delay between pulls |
| `order` | string | `random` | Pull order: `random`, `alphabetic`, `smallestFirst`, `largestFirst`, `oldestFirst`, `newestFirst` |
| `blockPullOrder` | string | `standard` | Block download order: `standard`, `random`, `inOrder` |

### Filesystem Options

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `ignorePerms` | bool | `false` | Ignore permission differences |
| `ignoreDelete` | bool | `false` | Don't delete files removed on remote |
| `autoNormalize` | bool | `true` | Auto-normalize Unicode filenames |
| `caseSensitiveFS` | bool | `false` | Force case-sensitive behavior |
| `disableSparseFiles` | bool | `false` | Disable sparse file creation |
| `disableFsync` | bool | `false` | Skip fsync (faster but risky on crash) |
| `minDiskFree` | object | — | Minimum free disk space (`{"value": 1, "unit": "%"}`) |
| `maxConflicts` | int | `10` | Max conflict files to keep (-1 = unlimited, 0 = none) |
| `copyRangeMethod` | string | — | Optimize for network filesystems |

### Metadata Sync

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `syncOwnership` | bool | `false` | Sync file ownership |
| `sendOwnership` | bool | `false` | Send ownership to peers |
| `syncXattrs` | bool | `false` | Sync extended attributes |
| `sendXattrs` | bool | `false` | Send xattrs to peers |
| `xattrFilter` | object | — | Xattr permit/deny patterns |

### Versioning Field

| Field | Type | Description |
|-------|------|-------------|
| `versioning` | object | Versioning strategy (see [Versioning](#versioning) section) |

---

## Device Configuration

Retrieved via `GET /rest/config/devices/{id}`.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `deviceID` | string | — | Unique device ID (SHA-256 fingerprint, required) |
| `name` | string | — | Human-readable name |
| `addresses` | array | `["dynamic"]` | Connection endpoints: `dynamic`, `tcp://IP:PORT`, `quic://IP:PORT` |
| `compression` | string | `metadata` | Compression: `metadata`, `always`, `never` |
| `paused` | bool | `false` | Pause synchronization with this device |
| `introducer` | bool | `false` | Trust as introducer (auto-adds its peers) |
| `skipIntroductionRemovals` | bool | `false` | Don't remove devices when introducer removes them |
| `introducedBy` | string | — | Device ID of the introducer |
| `autoAcceptFolders` | bool | `false` | Auto-accept folders shared by this device |
| `untrusted` | bool | `false` | Encrypted-only communication |
| `maxSendKbps` | int | `0` | Per-device upload limit (0 = unlimited) |
| `maxRecvKbps` | int | `0` | Per-device download limit (0 = unlimited) |
| `maxRequestKiB` | int | `0` | Max request size in KiB |
| `allowedNetworks` | array | `[]` | Networks allowed to connect |
| `numConnections` | int | `3` | Number of parallel connections |
| `ignoredFolders` | array | `[]` | Folder IDs to ignore from this device |
| `certName` | string | — | Certificate name override |

### Introducer Behavior

When a device is marked as introducer:
- It automatically adds devices it knows about to mutually-shared folders
- Only introduces devices sharing mutual folders
- Avoid mutual introducers (both sides introducing each other) to prevent loops

---

## GUI Configuration

Retrieved via `GET /rest/config/gui`.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `enabled` | bool | `true` | Enable web GUI and REST API |
| `address` | string | `127.0.0.1:8384` | Listen address |
| `tls` | bool | `false` | Enforce HTTPS |
| `apikey` | string | (generated) | REST API key |
| `authMode` | string | `static` | Authentication: `static` or `ldap` |
| `user` | string | — | Username for static auth |
| `password` | string | — | Bcrypt-hashed password |
| `theme` | string | `default` | UI theme |
| `insecureAllowFrameLoading` | bool | `false` | Allow iframe embedding |
| `insecureSkipHostcheck` | bool | `false` | Skip Host header validation |
| `insecureAdminAccess` | bool | `false` | Allow non-localhost without password |

**Security note:** The web GUI is restricted to localhost by default. Enabling `insecureSkipHostcheck` or `insecureAdminAccess` on a public network is a security risk.

---

## Options (Global Settings)

Retrieved via `GET /rest/config/options`.

### Network

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `listenAddresses` | array | `["default"]` | Sync protocol listen addresses (TCP/QUIC) |
| `globalAnnounceEnabled` | bool | `true` | Use global discovery servers |
| `globalAnnounceServers` | array | `["default"]` | Discovery server URLs |
| `localAnnounceEnabled` | bool | `true` | Use local discovery (port 21027/UDP) |
| `relaysEnabled` | bool | `true` | Enable relay fallback |
| `relayServers` | array | `["default"]` | Relay server URLs |

### NAT Traversal

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `natEnabled` | bool | `true` | Enable UPnP/NAT-PMP |
| `natLeaseMinutes` | int | `60` | NAT lease duration |
| `natRenewalMinutes` | int | `30` | NAT renewal interval |
| `stunServers` | array | `["default"]` | STUN servers |
| `stunKeepaliveStartS` | int | `180` | STUN keepalive start |
| `stunKeepaliveMinS` | int | `20` | STUN keepalive minimum |

### Bandwidth

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `maxSendKbps` | int | `0` | Global upload limit (0 = unlimited) |
| `maxRecvKbps` | int | `0` | Global download limit (0 = unlimited) |
| `limitBandwidthInLan` | bool | `false` | Apply limits on LAN |

### Upgrades

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `autoUpgradeIntervalH` | int | `12` | Upgrade check interval (0 = disable) |
| `upgradeToPreReleases` | bool | `false` | Include pre-release versions |

### Performance

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `maxFolderConcurrency` | int | `0` | Concurrent I/O operations (0 = auto) |
| `maxConcurrentIncomingRequestKiB` | int | `0` | Limit incoming request rate |
| `cacheIgnoredFiles` | bool | `false` | Cache ignore pattern results |

### Telemetry

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `urAccepted` | int | `0` | Usage reporting consent (-1 = declined, 0 = undecided, 3 = accepted) |
| `crashReportingEnabled` | bool | `true` | Send crash reports |

---

## LDAP Configuration

Retrieved via `GET /rest/config/ldap`. Used when `gui.authMode` is `ldap`.

| Field | Type | Description |
|-------|------|-------------|
| `address` | string | LDAP server (e.g., `dc1.example.com:389`) |
| `bindDN` | string | Bind pattern with `%s` for username |
| `transport` | string | `plain`, `tls`, or `starttls` |
| `searchBaseDN` | string | Search base DN |
| `searchFilter` | string | LDAP search filter |

---

## Versioning

Configured per-folder in the `versioning` field. All strategies store versions in `.stversions` by default (configurable via `fsPath`).

### Trash Can

Moves old files to `.stversions`. Only keeps the most recent replaced version.

```json
{
  "type": "trashcan",
  "params": {
    "cleanoutDays": "30",
    "fsPath": ""
  },
  "cleanupIntervalS": 3600
}
```

### Simple

Keeps N timestamped copies per file.

```json
{
  "type": "simple",
  "params": {
    "keep": "5",
    "cleanoutDays": "0",
    "fsPath": ""
  },
  "cleanupIntervalS": 3600
}
```

### Staggered

Tiered retention with automatic thinning:
- Last hour: one version per 30 seconds
- Last day: one version per hour
- Last 30 days: one version per day
- Beyond: one version per week

```json
{
  "type": "staggered",
  "params": {
    "maxAge": "365",
    "fsPath": ""
  },
  "cleanupIntervalS": 3600
}
```

`maxAge`: Days to keep versions (0 = forever).

### External

Delegates versioning to a custom script. The script receives file paths as arguments.

```json
{
  "type": "external",
  "params": {
    "command": "/path/to/script %FOLDER_PATH% %FILE_PATH%"
  }
}
```

Available variables: `%FOLDER_PATH%`, `%FILE_PATH%`.

---

## Ignore Patterns

Patterns to exclude files from synchronization. Stored in `.stignore` at the folder root, or managed via the API.

### API Access

```bash
# Get patterns
GET /rest/db/ignores?folder=FOLDER_ID

# Set patterns
POST /rest/db/ignores?folder=FOLDER_ID
Body: {"ignore": ["pattern1", "pattern2"]}
```

### Pattern Syntax

| Pattern | Meaning |
|---------|---------|
| `*.tmp` | Match files ending in .tmp |
| `!important.tmp` | Negation — un-ignore this file |
| `(?i)Thumbs.db` | Case-insensitive match |
| `(?d)temp/` | Mark as deletable (can be removed by Syncthing) |
| `/pattern` | Match in current directory only |
| `**/pattern` | Match in any subdirectory |
| `pattern/` | Equivalent to `pattern/**` |
| `#include filename` | Include patterns from another file |

### Evaluation Rules

- Patterns are evaluated top-to-bottom, first match wins
- Lines starting with `//` are comments
- A pattern without `/` prefix matches in current directory and all subdirectories
- Trailing `/` matches directory contents recursively

### Common Patterns

```
// macOS
.DS_Store
._*

// Windows
Thumbs.db
desktop.ini

// Development
node_modules
.git
*.pyc
__pycache__

// Temporary files
*.tmp
*.swp
~*
```

---

## Defaults

Templates for newly created folders and devices.

```bash
# Get/set default folder template
GET  /rest/config/defaults/folder
PUT  /rest/config/defaults/folder

# Get/set default device template
GET  /rest/config/defaults/device
PUT  /rest/config/defaults/device

# Get/set default ignore patterns
GET  /rest/config/defaults/ignores
PUT  /rest/config/defaults/ignores
```

These templates are applied when a new folder or device is added without specifying all fields.
