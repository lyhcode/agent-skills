# Syncthing REST API Reference

All endpoints are prefixed with `http://127.0.0.1:8384` (default).
Authentication required unless noted. Use `X-API-Key` header or `Authorization: Bearer <key>`.

## Table of Contents

- [System Endpoints](#system-endpoints)
- [Config Endpoints](#config-endpoints)
- [Database Endpoints](#database-endpoints)
- [Folder Endpoints](#folder-endpoints)
- [Cluster Endpoints](#cluster-endpoints)
- [Event Endpoints](#event-endpoints)
- [Statistics Endpoints](#statistics-endpoints)
- [Service Utility Endpoints](#service-utility-endpoints)
- [Debug Endpoints](#debug-endpoints)
- [No-Auth Endpoints](#no-auth-endpoints)

---

## System Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/rest/system/status` | System status: device ID, uptime, memory, goroutines |
| GET | `/rest/system/version` | Version, OS, architecture, build info |
| GET | `/rest/system/ping` | Health check (returns `{"ping": "pong"}`) |
| POST | `/rest/system/ping` | Same as GET |
| GET | `/rest/system/connections` | Connection info for all devices (connected, address, type, crypto) |
| GET | `/rest/system/browse` | Browse filesystem directories. Param: `current` (path) |
| GET | `/rest/system/discovery` | Cached device discovery results |
| POST | `/rest/system/discovery` | Register device at address. Params: `device`, `addr` |
| GET | `/rest/system/error` | System error log |
| POST | `/rest/system/error` | Report an external error. Body: `{"message": "..."}` |
| POST | `/rest/system/error/clear` | Clear all errors |
| GET | `/rest/system/log` | JSON log entries. Param: `since` (timestamp) |
| GET | `/rest/system/log.txt` | Plain-text log |
| GET | `/rest/system/loglevels` | Current log level per facility |
| POST | `/rest/system/loglevels` | Set log levels. Body: `{"facilities": {"api": 5}}` |
| GET | `/rest/system/paths` | Configuration directory paths |
| POST | `/rest/system/pause` | Pause sync. Optional param: `device` (specific device) |
| POST | `/rest/system/resume` | Resume sync. Optional param: `device` (specific device) |
| POST | `/rest/system/reset` | Reset database. Optional param: `folder` (specific folder) |
| POST | `/rest/system/restart` | Restart Syncthing |
| POST | `/rest/system/shutdown` | Graceful shutdown |
| GET | `/rest/system/upgrade` | Check for available upgrade |
| POST | `/rest/system/upgrade` | Download and apply upgrade |

---

## Config Endpoints

These endpoints manage the live configuration. Changes take effect immediately (some require restart — check `/rest/config/restart-required`).

### Full Configuration

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/rest/config` | Get entire configuration |
| PUT | `/rest/config` | Replace entire configuration |
| GET | `/rest/config/restart-required` | Check if restart needed after changes |

### Folders

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/rest/config/folders` | List all folders |
| PUT | `/rest/config/folders` | Replace all folders |
| POST | `/rest/config/folders` | Add a new folder |
| GET | `/rest/config/folders/{id}` | Get specific folder |
| PUT | `/rest/config/folders/{id}` | Replace folder config |
| PATCH | `/rest/config/folders/{id}` | Partial update folder |
| DELETE | `/rest/config/folders/{id}` | Delete folder |

### Devices

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/rest/config/devices` | List all devices |
| PUT | `/rest/config/devices` | Replace all devices |
| POST | `/rest/config/devices` | Add a new device |
| GET | `/rest/config/devices/{id}` | Get specific device |
| PUT | `/rest/config/devices/{id}` | Replace device config |
| PATCH | `/rest/config/devices/{id}` | Partial update device |
| DELETE | `/rest/config/devices/{id}` | Delete device |

### Defaults

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/rest/config/defaults/folder` | Default folder template |
| PUT | `/rest/config/defaults/folder` | Set default folder template |
| PATCH | `/rest/config/defaults/folder` | Partial update default folder |
| GET | `/rest/config/defaults/device` | Default device template |
| PUT | `/rest/config/defaults/device` | Set default device template |
| PATCH | `/rest/config/defaults/device` | Partial update default device |
| GET | `/rest/config/defaults/ignores` | Default ignore patterns |
| PUT | `/rest/config/defaults/ignores` | Set default ignore patterns |

### Options, GUI, LDAP

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/rest/config/options` | Global options |
| PUT | `/rest/config/options` | Replace options |
| PATCH | `/rest/config/options` | Partial update options |
| GET | `/rest/config/gui` | GUI/API configuration |
| PUT | `/rest/config/gui` | Replace GUI config |
| PATCH | `/rest/config/gui` | Partial update GUI config |
| GET | `/rest/config/ldap` | LDAP configuration |
| PUT | `/rest/config/ldap` | Replace LDAP config |
| PATCH | `/rest/config/ldap` | Partial update LDAP |

**PATCH behavior:** Merges the JSON object on top of existing config. Replaces child objects entirely (does not extend arrays).

---

## Database Endpoints

| Method | Endpoint | Parameters | Description |
|--------|----------|------------|-------------|
| GET | `/rest/db/browse` | `folder` (required), `prefix`, `dirsonly`, `levels` | Browse folder directory structure |
| GET | `/rest/db/completion` | `folder`, `device` (both optional) | Sync completion percentage |
| GET | `/rest/db/file` | `folder` (required), `file` (required) | File info: global state, local state, availability |
| GET | `/rest/db/ignores` | `folder` (required) | Current ignore patterns |
| POST | `/rest/db/ignores` | `folder` (required), body: `{"ignore": [...]}` | Set ignore patterns |
| GET | `/rest/db/localchanged` | `folder` (required), `page`, `perpage` | Files changed locally (receive-only folders) |
| GET | `/rest/db/need` | `folder` (required), `page`, `perpage` | Files needing sync |
| GET | `/rest/db/remoteneed` | `folder` (required), `device` (required), `page`, `perpage` | Files a remote device needs |
| GET | `/rest/db/status` | `folder` (required) | Folder database status (state, files, bytes, errors) |
| POST | `/rest/db/override` | `folder` (required) | Override remote changes with local (send-only) |
| POST | `/rest/db/prio` | `folder` (required), `file` (required) | Prioritize file download |
| POST | `/rest/db/revert` | `folder` (required) | Revert local changes to remote (receive-only) |
| POST | `/rest/db/scan` | `folder` (required), `sub` (optional subdirectory) | Trigger folder rescan |

---

## Folder Endpoints

| Method | Endpoint | Parameters | Description |
|--------|----------|------------|-------------|
| GET | `/rest/folder/errors` | `folder` (required), `page`, `perpage` | Sync errors for a folder |
| GET | `/rest/folder/versions` | `folder` (required) | File version history |
| POST | `/rest/folder/versions` | `folder` (required), body: `{"file": "timestamp"}` | Restore file versions |

---

## Cluster Endpoints

| Method | Endpoint | Parameters | Description |
|--------|----------|------------|-------------|
| GET | `/rest/cluster/pending/devices` | — | Devices awaiting acceptance |
| DELETE | `/rest/cluster/pending/devices` | `device` (required) | Dismiss pending device |
| GET | `/rest/cluster/pending/folders` | `device` (required) | Folders awaiting acceptance |
| DELETE | `/rest/cluster/pending/folders` | `device` (required), `folder` (required) | Dismiss pending folder |

---

## Event Endpoints

Long-polling endpoints for real-time event streaming.

| Method | Endpoint | Parameters | Description |
|--------|----------|------------|-------------|
| GET | `/rest/events` | `since` (last event ID), `events` (comma-separated types), `limit`, `timeout` (default 60s) | Event stream |
| GET | `/rest/events/disk` | `since`, `limit`, `timeout` | Disk-related events only |

### Event Types

**Device:** DeviceConnected, DeviceDisconnected, DeviceDiscovered, DevicePaused, DeviceResumed

**Folder:** FolderCompletion, FolderErrors, FolderPaused, FolderResumed, FolderScanProgress, FolderSummary, FolderWatchStateChanged

**Sync:** ItemStarted, ItemFinished, LocalChangeDetected, RemoteChangeDetected, LocalIndexUpdated, RemoteIndexUpdated, RemoteDownloadProgress

**Cluster:** ClusterConfigReceived, PendingDevicesChanged, PendingFoldersChanged

**System:** ConfigSaved, Failure, LoginAttempt, Starting, StartupComplete, StateChanged, DownloadProgress, ListenAddressesChanged

### Event Structure

```json
{
  "id": 123,
  "globalID": 456,
  "type": "DeviceConnected",
  "time": "2024-01-15T10:30:00.000Z",
  "data": { ... }
}
```

---

## Statistics Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/rest/stats/device` | Per-device statistics (last seen, connection duration) |
| GET | `/rest/stats/folder` | Per-folder statistics (last scan, last file) |

---

## Service Utility Endpoints

| Method | Endpoint | Parameters | Description |
|--------|----------|------------|-------------|
| GET | `/rest/svc/deviceid` | `id` (device ID string) | Format/validate device ID |
| GET | `/rest/svc/lang` | — | Supported languages |
| GET | `/rest/svc/report` | — | Anonymous usage report data |
| GET | `/rest/svc/random/string` | `length` (default 32) | Generate random string |

---

## Debug Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/rest/debug/cpuprof` | Download CPU profile |
| GET | `/rest/debug/heapprof` | Download heap profile |
| GET | `/rest/debug/support` | Generate support bundle |
| GET | `/rest/debug/file` | Debug file access |

---

## No-Auth Endpoints

These do not require authentication:

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/rest/noauth/health` | Health check (`{"status": "OK"}`) |
| POST | `/rest/noauth/auth/password` | Password authentication |
| POST | `/rest/noauth/auth/logout` | Logout |

---

## Metrics

Prometheus-format metrics available at `/metrics` on the GUI/API address. Requires authentication if GUI does.

Metric categories: events, filesystem operations, folder sync progress, protocol traffic, scanner stats.
