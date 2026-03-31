---
name: synology-cli
description: >
  Manage Synology NAS via SSH using administrative CLI utilities at /usr/syno/sbin/.
  Use when the user wants to manage local users, groups, shared folders, network settings,
  services, or workgroup/domain settings on a Synology DiskStation.
  Triggers on mentions of Synology, DiskStation, NAS administration, synouser, synogroup,
  synoshare, synonet, synoservice, or synowin.
version_target: "DSM 7.x"
references:
  - https://global.download.synology.com/download/Document/Software/DeveloperGuide/Firmware/DSM/All/enu/Synology_DiskStation_Administration_CLI_Guide.pdf
---

# Synology NAS CLI Administration

Help users manage Synology DiskStation via SSH using the six CLI utilities located at `/usr/syno/sbin/`.

Reference: [Synology CLI Administrator Guide](https://global.download.synology.com/download/Document/Software/DeveloperGuide/Firmware/DSM/All/enu/Synology_DiskStation_Administration_CLI_Guide.pdf)

## Prerequisites

- SSH access to the Synology NAS (enable via DSM > Control Panel > Terminal & SNMP)
- Root/super-user privileges (all utilities require super-user)
- Commands are at `/usr/syno/sbin/` — use full path or add to `$PATH`

## User Management — synouser

```bash
# Add user (username passwd full_name expired email app_privilege)
/usr/syno/sbin/synouser --add john "p@ssw0rd" "John Doe" 0 john@example.com 0

# Delete user
/usr/syno/sbin/synouser --del john

# Rename user
/usr/syno/sbin/synouser --rename oldname newname

# Modify user info
/usr/syno/sbin/synouser --modify john "newpass" "John D." 0 john@example.com 0

# Set password only
/usr/syno/sbin/synouser --setpw admin newpassword
```

**Parameters:**
- `username`: 1–64 UTF-8 chars, not case sensitive
- `passwd`: up to 127 chars, case sensitive
- `expired`: 0 = active, 1 = expired
- `app_privilege`: bitmask — FTP (0x01), File Station (0x02), Audio Station (0x04), Download Station (0x08), Surveillance Station (0x10). Sum for multiple (e.g., 31 = all)

## Group Management — synogroup

```bash
# Add group with initial members
/usr/syno/sbin/synogroup --add developers user1 user2

# Delete group
/usr/syno/sbin/synogroup --del developers

# Rename group
/usr/syno/sbin/synogroup --rename oldgroup newgroup

# Set group members (replaces entire member list)
/usr/syno/sbin/synogroup --member developers user1 user2 user3
```

**Restrictions:** Group name 1–15 UTF-8 chars. System groups cannot be deleted or renamed.

## Shared Folder Management — synoshare

```bash
# Add shared folder
# synoshare --add name desc path user_list_na user_list_ro user_list_rw browsable adv_privilege
/usr/syno/sbin/synoshare --add projects "Projects" /volume1/projects "" "" "" 1 0

# Delete shared folder (TRUE = delete data, FALSE = config only)
/usr/syno/sbin/synoshare --del TRUE projects

# Rename shared folder
/usr/syno/sbin/synoshare --rename oldname newname

# Set user permissions (NA|RO|RW) (+|-|=) user_list
/usr/syno/sbin/synoshare --setuser projects RW + user1,user2,@group1
/usr/syno/sbin/synoshare --setuser projects RO = user3
/usr/syno/sbin/synoshare --setuser projects NA - user4
```

**Permission operators:** `+` append, `-` remove, `=` replace. Prefix groups with `@`.

**adv_privilege** bitmask: disable browsing (0x1), disable modification (0x2), disable download (0x4).

**Reserved names:** global, homes, home, printers, surveillance, usbbackup, usbshare, esatashare.

## Network Settings — synonet

```bash
# Set DHCP
/usr/syno/sbin/synonet --dhcp eth0

# Set static IP
/usr/syno/sbin/synonet --manual eth0 192.168.1.100 255.255.255.0
/usr/syno/sbin/synonet --set_gateway 192.168.1.1
/usr/syno/sbin/synonet --set_dns 8.8.8.8

# Set MTU (1500–9000, increments of 1000)
/usr/syno/sbin/synonet --set_mtu eth0 9000

# Set hostname
/usr/syno/sbin/synonet --set_hostname mynas
```

**Restrictions:** Interface limited to `eth0` or `eth1`. IPs must be IPv4 format. Hostname 1–15 chars, must start with a letter.

## Service Management — synoservice

```bash
# List all services (or only running)
/usr/syno/sbin/synoservice --list
/usr/syno/sbin/synoservice --list running

# Enable/disable (starts/stops immediately)
/usr/syno/sbin/synoservice --enable ssh
/usr/syno/sbin/synoservice --disable telnet

# Start/stop/restart (without changing settings)
/usr/syno/sbin/synoservice --start ssh
/usr/syno/sbin/synoservice --stop ftp
/usr/syno/sbin/synoservice --restart samba

# Enable/disable settings only (no immediate start/stop)
/usr/syno/sbin/synoservice --keyon ssh
/usr/syno/sbin/synoservice --keyoff telnet

# Show service details
/usr/syno/sbin/synoservice --detail ssh
```

**Available services:** web, photo, netbkp, download, media, audio, itunes, mysql, printer, surveillance, userhome, ftp, telnet, ssh, nfs, afp, samba, filestation, https.

## Workgroup / ADS Domain — synowin

```bash
# Join workgroup (leaves ADS domain)
/usr/syno/sbin/synowin --joinWorkgroup WORKGROUP

# Join ADS domain
/usr/syno/sbin/synowin --joinDomain example.com admin password \
  -d 8.8.8.8 -i 10.0.0.1,10.0.0.2 -n EXAMPLE -f example.com
```

**joinDomain options:** `-d` DNS IP, `-i` KDC IP(s) (comma-separated, `*` after last for fallback), `-n` NetBIOS name, `-f` FQDN.

## Common Patterns

### Initial NAS setup via SSH

```bash
# Set hostname and network
/usr/syno/sbin/synonet --set_hostname mynas
/usr/syno/sbin/synonet --manual eth0 192.168.1.100 255.255.255.0
/usr/syno/sbin/synonet --set_gateway 192.168.1.1
/usr/syno/sbin/synonet --set_dns 8.8.8.8

# Enable SSH and disable telnet
/usr/syno/sbin/synoservice --enable ssh
/usr/syno/sbin/synoservice --disable telnet

# Create user and shared folder
/usr/syno/sbin/synouser --add dev "password" "Developer" 0 dev@example.com 3
/usr/syno/sbin/synoshare --add projects "Projects" /volume1/projects "" "" "dev" 1 0
```

### Manage team access

```bash
# Create group and add members
/usr/syno/sbin/synogroup --add team user1 user2 user3

# Grant group read/write access to shared folder
/usr/syno/sbin/synoshare --setuser data RW + @team
```

## Important Notes

- All commands require root privileges (`sudo` or root SSH login)
- Exit status: 0 = success, >0 = error
- Changes take effect immediately — use caution with network settings as you may lose SSH connectivity
- `--enable`/`--disable` both saves settings AND starts/stops the service; use `--keyon`/`--keyoff` to only save settings without affecting the running service
