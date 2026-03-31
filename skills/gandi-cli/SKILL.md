---
name: gandi-cli
description: >
  Manage Gandi.net domains, DNS records, certificates, and email using the gandi CLI.
  Use when the user wants to manage domain names, LiveDNS records, SSL certificates,
  email forwarding, mailboxes, or organizations on Gandi.net.
  Triggers on mentions of Gandi, domain management, DNS records, or gandi-cli.
version_target: "agent-gandi-cli 0.1"
references:
  - https://github.com/lyhcode/agent-gandi-cli
  - https://api.gandi.net/docs/
  - https://api.gandi.net/docs/livedns/
---

# Gandi CLI Assistant

Help users manage Gandi.net services using the `gandi` CLI.

Source: https://github.com/lyhcode/agent-gandi-cli

## Prerequisites

- Python 3.11+ with `uvx` or `pip`
- A Gandi Personal Access Token (PAT) from https://account.gandi.net/

## Installation & Usage

Run directly with `uvx` (no install needed):

```bash
uvx agent-gandi-cli gandi <command>
```

Or install globally:

```bash
pip install agent-gandi-cli
gandi <command>
```

## Authentication

Token priority: `--token` flag > `GANDI_PAT` env var > config file.

```bash
# Save token to config file
gandi auth login --token <your-pat>

# Or set environment variable
export GANDI_PAT="your-personal-access-token"

# Check authentication status
gandi auth status

# Remove saved token
gandi auth logout

# Set default organization
gandi auth set-org <sharing-id>
```

Config file location: `~/.config/gandi-cli/config.toml` (TOML format)

```toml
[auth]
pat = "your-token"

[defaults]
sharing_id = ""
output = "table"
```

## Global Options

| Option | Description |
|--------|-------------|
| `--output`, `-o` | Output format: `table` (default), `json`, `plain` |
| `--token` | API token (overrides config, env: `GANDI_PAT`) |
| `--sharing-id` | Organization ID (env: `GANDI_SHARING_ID`) |
| `--sandbox` | Use sandbox API (`api.sandbox.gandi.net`) |

## Domain Management

```bash
# List domains
gandi domain list
gandi domain list --fqdn "example"       # Filter by FQDN pattern
gandi domain list --per-page 50          # Pagination

# Domain details
gandi domain info example.com

# Check availability and pricing
gandi domain check newdomain.com
gandi domain check newdomain.com --currency TWD
```

## DNS Record Management (LiveDNS)

```bash
# List all records
gandi dns list example.com
gandi dns list example.com --type A               # Filter by type
gandi dns list example.com --name www              # Filter by name

# Get specific record
gandi dns get example.com www A
gandi dns get example.com @ MX

# Create record
gandi dns create example.com www A 1.2.3.4
gandi dns create example.com www A 1.2.3.4 5.6.7.8    # Multiple values
gandi dns create example.com mail MX "10 mail.example.com." --ttl 3600
gandi dns create example.com @ TXT "v=spf1 include:_spf.google.com ~all"

# Update record
gandi dns update example.com www A --value 9.8.7.6
gandi dns update example.com www A --value 1.1.1.1 --value 2.2.2.2 --ttl 600

# Delete record
gandi dns delete example.com www A
gandi dns delete example.com www A --force          # Skip confirmation

# Export zone file
gandi dns export example.com
```

### Common DNS Record Types

| Type | Example Value | Description |
|------|---------------|-------------|
| `A` | `1.2.3.4` | IPv4 address |
| `AAAA` | `2001:db8::1` | IPv6 address |
| `ALIAS` | `alias.example.com.` | Alias for bare domain (like CNAME on @) |
| `CNAME` | `alias.example.com.` | Canonical name (trailing dot) |
| `MX` | `10 mail.example.com.` | Mail exchange (priority + host) |
| `TXT` | `"v=spf1 ..."` | Text record (SPF, DKIM, verification) |
| `NS` | `ns1.example.com.` | Nameserver |
| `SRV` | `10 5 5060 sip.example.com.` | Service record |
| `CAA` | `0 issue "letsencrypt.org"` | Certificate authority authorization |

All supported types: A, AAAA, ALIAS, CAA, CDS, CNAME, DNAME, DS, HTTPS, KEY, LOC, MX, NAPTR, NS, OPENPGPKEY, PTR, RP, SOA, SPF, SRV, SSHFP, SVCB, TLSA, TXT, WKS

### Important Constraints

- **ALIAS + DNSSEC**: ALIAS records on bare domain (`@`) will break DNSSEC — avoid combining them
- **CNAME exclusivity**: CNAME cannot coexist with other record types on the same name
- **Email forwards**: Max 100 creations/updates per week per domain (HTTP 429 on exceed)

## Certificate Management

```bash
# List SSL certificates
gandi cert list
gandi cert list --cn example.com              # Filter by Common Name
gandi cert list --status valid                # Filter: valid, expired, pending, revoked

# Certificate details
gandi cert info <cert-id>
```

## Email Management

### Email Forwarding

```bash
# List forwards
gandi email forward list example.com

# Create forward
gandi email forward create example.com info user@gmail.com
gandi email forward create example.com sales user1@gmail.com user2@gmail.com

# Delete forward
gandi email forward delete example.com info
gandi email forward delete example.com info --force
```

### Mailboxes

```bash
# List mailboxes
gandi email mailbox list example.com

# Mailbox details
gandi email mailbox info example.com <mailbox-id>
```

## Organization Management

```bash
# List organizations
gandi org list
gandi org list --name "My Company"

# Organization details
gandi org info <org-id>

# Show current user
gandi org whoami
```

## Output Formats

```bash
# Table output (default, human-readable)
gandi domain list

# JSON output (for scripting)
gandi domain list -o json

# Plain output (minimal)
gandi domain list -o plain

# Pipe JSON to jq
gandi dns list example.com -o json | jq '.[].rrset_name'
```

## Common Patterns

### Set up a new subdomain

```bash
gandi dns create example.com blog A 1.2.3.4
gandi dns create example.com blog AAAA 2001:db8::1
```

### Update DNS for a migration

```bash
gandi dns update example.com www A --value 10.0.0.1 --ttl 300
# After migration is verified, restore normal TTL
gandi dns update example.com www A --value 10.0.0.1 --ttl 10800
```

### Add email forwarding

```bash
gandi email forward create example.com info personal@gmail.com
gandi email forward create example.com support team@company.com
```

### Export DNS for backup

```bash
gandi dns export example.com > example.com.zone
```

## Shell Completion

```bash
# Install completion for current shell
gandi --install-completion

# Show completion script
gandi --show-completion
```
