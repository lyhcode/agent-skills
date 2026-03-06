# 1Password CLI Command Reference

Full command reference for `op` CLI. Source: https://developer.1password.com/docs/cli/reference

## Management Commands

| Command | Description |
|---------|-------------|
| `op account` | Configure and manage locally set up 1Password accounts |
| `op connect` | Administer Connect server instances and authentication tokens |
| `op document` | CRUD operations on Document items |
| `op environment` | Administer 1Password Environments and variables (Beta) |
| `op events-api` | Control Events API integrations |
| `op group` | Manage groups in your 1Password account |
| `op item` | CRUD operations on vault items |
| `op plugin` | Manage shell plugins for authenticating third-party CLIs |
| `op service-account` | Manage service accounts |
| `op user` | Manage users in your 1Password account |
| `op vault` | Manage permissions and CRUD operations on vaults |

## Utility Commands

| Command | Description |
|---------|-------------|
| `op completion` | Generate shell completion scripts |
| `op inject` | Inject secrets into config file templates |
| `op read` | Read a single secret reference |
| `op run` | Pass secrets as environment variables to a subprocess |
| `op signin` | Authenticate to a 1Password account |
| `op signout` | End authentication session |
| `op update` | Check for and install updates |
| `op whoami` | Get info about the authenticated account |

## Item Subcommands

| Command | Description |
|---------|-------------|
| `op item create` | Create a new item |
| `op item delete` | Delete or archive an item |
| `op item edit` | Edit an existing item |
| `op item get` | Get details of an item |
| `op item list` | List items in a vault |
| `op item move` | Move an item between vaults |
| `op item share` | Share an item |
| `op item template get` | Get a JSON template for a category |
| `op item template list` | List all available item categories |

### op item create

```
op item create [ - ] [ <assignment>... ] [flags]
```

**Flags:**
- `--category <category>` — Item category (case-insensitive, ignores whitespace)
- `--title <title>` — Item name
- `--vault <vault>` — Target vault (defaults to Personal/Private/Employee vault)
- `--url <url>` — Website for autofill (Login, Password, API Credential)
- `--generate-password[=<recipe>]` — Generate password (recipe: `letters,digits,symbols,<length>`)
- `--tags <tag1,tag2>` — Comma-separated tags
- `--template <path>` — JSON template file path
- `--dry-run` — Preview resulting item without creating
- `--favorite` — Add item to favorites
- `--ssh-generate-key <type>` — Generate SSH key (`ed25519`, `rsa`, `rsa2048`, `rsa3072`, `rsa4096`)
- `--reveal` — Don't conceal sensitive fields in output

### op item get

```
op item get [{ <itemName> | <itemID> | <shareLink> | - }] [flags]
```

**Flags:**
- `--fields <field-list>` — Get specific fields (`label=<name>` or `type=<type>`, comma-separated)
- `--format json` — JSON output
- `--vault <vault>` — Specify vault
- `--otp` — Output the primary one-time password
- `--share-link` — Get a shareable link for the item
- `--include-archive` — Include archived items
- `--reveal` — Don't conceal sensitive fields

### op item edit

```
op item edit { <itemName> | <itemID> | <shareLink> } [ <assignment>... ] [flags]
```

**Flags:**
- `--title <title>` — Change title
- `--vault <vault>` — Move to a different vault
- `--url <url>` — Change URL
- `--tags <tags>` — Replace tags (empty value removes all)
- `--generate-password[=<recipe>]` — Generate new password
- `--template <path>` — JSON template file
- `--dry-run` — Preview resulting item without saving
- `--favorite true|false` — Set favorite status
- `--reveal` — Don't conceal sensitive fields

**Delete a field:** Use `[delete]` as fieldType: `'Field Name[delete]'`

**Cannot edit:** SSH keys via `op item edit`. JSON templates do not support passkeys (will overwrite).

### op item delete

```
op item delete [{ <itemName> | <itemID> | <shareLink> | - }] [flags]
```

**Flags:**
- `--archive` — Move to Archive instead of delete
- `--vault <vault>` — Specify vault

Deleted items remain in Recently Deleted for 30 days.

### op item move

```
op item move [{ <itemName> | <itemID> | <shareLink> | - }] [flags]
```

**Flags:**
- `--current-vault <vault>` — Vault where item currently lives
- `--destination-vault <vault>` — Target vault

Moving creates a copy in the destination and deletes the original (item gets a new ID).

### op item share

```
op item share { <itemName> | <itemID> } [flags]
```

**Flags:**
- `--emails <email1,email2>` — Restrict access to specific emails
- `--expires-in <duration>` — Link expiration (`s`/`m`/`h`/`d`/`w`, default `7d`)
- `--view-once` — Expire after single view
- `--vault <vault>` — Specify vault

File attachments and Document items cannot be shared.

### op item list

```
op item list [flags]
```

**Flags:**
- `--categories <cat1,cat2>` — Filter by categories
- `--tags <tag1,tag2>` — Filter by tags
- `--vault <vault>` — Filter by vault
- `--favorite` — Only favorites
- `--include-archive` — Include archived items
- `--long` — Detailed output

## Vault Subcommands

| Command | Description |
|---------|-------------|
| `op vault create <name>` | Create a new vault |
| `op vault delete <vault>` | Delete a vault |
| `op vault edit <vault>` | Edit name, description, icon, Travel Mode |
| `op vault get <vault>` | Get vault details |
| `op vault list` | List all vaults |
| `op vault user grant` | Grant user access to a vault |
| `op vault user revoke` | Revoke user access from a vault |
| `op vault user list <vault>` | List users with access |
| `op vault group grant` | Grant group access to a vault |
| `op vault group revoke` | Revoke group access from a vault |
| `op vault group list <vault>` | List groups with access |

### Vault Permissions

**Teams:** `allow_viewing`, `allow_editing`, `allow_managing`

**Business (granular):**
- Viewing: `view_items`, `view_and_copy_passwords`, `view_item_history`
- Editing: `create_items`, `edit_items`, `archive_items`, `delete_items`, `import_items`, `export_items`, `copy_and_share_items`, `print_items`
- Managing: `manage_vault`

```bash
op vault user grant --vault=Staging --user="wendy@example.com" \
  --permissions=allow_viewing,allow_editing

op vault group grant --vault=Production --group="DevOps" \
  --permissions=view_items,create_items
```

### Vault Icons

Valid keywords: `airplane`, `application`, `art-supplies`, `bankers-box`, `brown-briefcase`, `brown-gate`, `buildings`, `cabin`, `castle`, `circle-of-dots`, `coffee`, `color-wheel`, `curtained-window`, `document`, `doughnut`, `fence`, `galaxy`, `gears`, `globe`, `green-backpack`, `green-gem`, `handshake`, `heart-with-monitor`, `house`, `id-card`, `jet`, `large-ship`, `luggage`, `plant`, `porthole`, `puzzle`, `rainbow`, `record`, `round-door`, `sandals`, `scales`, `screwdriver`, `shop`, `tall-window`, `treasure-chest`, `vault-door`, `vehicle`, `wallet`, `wrench`

## User Subcommands

| Command | Description |
|---------|-------------|
| `op user confirm <user>` | Confirm a user (supports `--all`) |
| `op user delete <user>` | Remove a user and all their data |
| `op user edit <user>` | Change name or Travel Mode (`--name`, `--travel-mode on\|off`) |
| `op user get <user>` | Get user details (`--me`, `--fingerprint`, `--public-key`) |
| `op user list` | List users (`--group`, `--vault` filters) |
| `op user provision` | Provision a user (`--name`, `--email`, `--language`) |
| `op user suspend <user>` | Suspend a user (logged out immediately) |
| `op user reactivate <user>` | Reactivate a suspended user |
| `op user recovery begin <user>` | Begin account recovery |

Users can be specified by email, name, or ID.

## Group Subcommands

| Command | Description |
|---------|-------------|
| `op group create <name>` | Create a new group |
| `op group delete <group>` | Delete a group |
| `op group edit <group>` | Edit a group |
| `op group get <group>` | Get group details |
| `op group list` | List all groups |
| `op group user grant` | Add a user to a group |
| `op group user revoke` | Remove a user from a group |
| `op group user list <group>` | List users in a group |

## Document Subcommands

| Command | Description |
|---------|-------------|
| `op document create <file>` | Upload a new document |
| `op document delete <doc>` | Delete a document |
| `op document edit <doc> <file>` | Replace a document's file |
| `op document get <doc>` | Download a document (`--out-file`) |
| `op document list` | List all documents |

## Plugin Subcommands

| Command | Description |
|---------|-------------|
| `op plugin list` | List all available shell plugins |
| `op plugin init <cli>` | Install and configure a plugin |
| `op plugin inspect <cli>` | View configured credentials and defaults |
| `op plugin run <command>` | Run command with provisioned credentials |
| `op plugin clear <cli>` | Clear configured defaults (`--all` for all scopes) |

### Plugin Credential Scopes

- **Terminal session** — only for current terminal
- **Directory** — for current directory and subdirectories
- **Global** — system-wide default

## Secret Reference Format

```
op://<vault>/<item>/[<section>/]<field>[?<query>]
```

- `vault`: Vault name or ID
- `item`: Item title or ID
- `section`: (Optional) Section name
- `field`: Field label
- `query`: Optional query parameters

### Query Parameters

| Parameter | Values | Example |
|-----------|--------|---------|
| `attribute` | `type`, `value`, `title`, `id`, `purpose`, `otp` | `?attribute=otp` |
| `ssh-format` | `openssh` | `?ssh-format=openssh` |

File attachment attributes: `content`, `size`, `id`, `name`, `type`

## Field Assignment Format

```
[<section>.]<field>[[<fieldType>]]=<value>
```

- For built-in fields: use field id from template, omit `fieldType`
- For custom fields: specify `fieldType` (defaults to `password` if omitted)
- Use `[delete]` to remove a custom field
- Escape `.` `=` `\` in section/field names with `\`

### Field Types

| Assignment `fieldType` | JSON `type` | Description |
|------------------------|-------------|-------------|
| `password` | `CONCEALED` | Concealed password (default for custom fields) |
| `text` | `STRING` | Text string |
| `email` | `EMAIL` | Email address |
| `url` | `URL` | Web address (for copying, not autofill) |
| `date` | `DATE` | Date (`YYYY-MM-DD`) |
| `monthYear` | `MONTH_YEAR` | Month/year (`YYYYMM` or `YYYY/MM`) |
| `phone` | `PHONE` | Phone number |
| `otp` | `OTP` | One-time password (`otpauth://` URI) |
| `file` | N/A | File attachment (assignment only, path as value) |

### Item Categories

API Credential, Bank Account, Credit Card, Database, Document, Driver License, Email Account, Identity, Login, Membership, Outdoor License, Passport, Password, Reward Program, Secure Note, Server, Social Security Number, Software License, Wireless Router

## Environment File (.env) Syntax

- `KEY=VALUE` statements separated by newlines
- Lines starting with `#` are comments (also inline after values)
- Values can be quoted with `"` or `'` (quotes are stripped)
- `$VAR_NAME` / `${VAR_NAME}` are expanded from environment
- Single-quoted values do NOT expand variables
- Empty lines are skipped, `EMPTY=` sets empty string
- Multi-line values supported within quotes
- UTF-8 encoding

## Common Flags

| Flag | Description |
|------|-------------|
| `--format=json` | Output in JSON format |
| `--vault=<name>` | Scope to a specific vault |
| `--categories=<cat>` | Filter by item category |
| `--tags=<tags>` | Filter by tags (comma-separated) |
| `--archive` | Archive instead of permanently delete |
| `--out-file=<path>` | Write output to file |
| `--no-masking` | Disable automatic secret masking (op run) |
| `--env-file=<path>` | Load environment file (op run) |
| `--template=<path>` | Use JSON template file |
| `--generate-password[=<recipe>]` | Generate password with optional recipe |

## Shell Completion

```bash
# Bash — add to .bashrc
source <(op completion bash)

# Zsh — add to .zshrc
eval "$(op completion zsh)"; compdef _op op

# fish — add to config.fish
op completion fish | source
```
