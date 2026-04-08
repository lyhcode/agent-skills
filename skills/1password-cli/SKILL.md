---
name: 1password-cli
description: >
  Manage 1Password items, vaults, and secrets using the op CLI.
  Use when the user wants to create, read, update, or delete passwords and secrets,
  manage vaults, inject secrets into config files, run commands with secret environment variables,
  or read secret references. Also use this skill when the user mentions .env secret management,
  credential rotation, vault organization, storing API tokens or SSH keys,
  or any workflow involving op://, op read, op run, or op inject.
  Triggers on mentions of 1Password, op CLI, password manager, secret management,
  credential storage, or secure configuration.
version_target: "op 2.33"
references:
  - https://developer.1password.com/docs/cli/
  - https://releases.1password.com/developers/cli/
---

# 1Password CLI (op) Assistant

Help users manage 1Password secrets and items using the `op` CLI.

## Prerequisites

- The `op` CLI must be installed (`brew install 1password-cli` on macOS)
- Desktop app integration must be enabled (1Password app > Settings > Developer > Integrate with 1Password CLI)
- Verify setup: `op whoami`

## Security Principles

Secrets that leak into terminal output, shell history, or version-controlled files are hard to revoke and easy to miss. These guidelines reduce that surface area:

1. **Avoid printing raw secrets** unless the user explicitly asks. Prefer `op read --out-file` or `op run` to keep secrets out of scrollback and logs.
2. **Use `--format=json`** for list/get commands — structured output is easier to parse and less likely to be misread.
3. **Use JSON templates** when creating/editing items with sensitive values. Inline assignment statements (`'password=...'`) can appear in `ps` output on shared machines.
4. **Prefer `op://` references** over reading secrets into shell variables. References are resolved just-in-time and never persist in memory.
5. **Keep secrets out of version control.** Don't write resolved values to files that might be committed. Use `.env` files with `op://` references instead.
6. **Delete template files** after use — they contain resolved secrets on disk.

## Reference Files

This skill includes two reference files for detailed lookups. Consult them as needed:

| File | When to use |
|------|-------------|
| [reference/commands.md](reference/commands.md) | Looking up specific flags, subcommands, field assignment syntax, or shell completion setup |
| [reference/conventions.md](reference/conventions.md) | Deciding item categories, naming items, organizing vaults, choosing field types, Markdown syntax limits in notes, or reviewing the best-practices checklist |

## Secret References (`op://`)

The `op://` URI is the core abstraction — it points to a secret without exposing it:

```
op://<vault>/<item>/[<section>/]<field>[?<query>]
```

- `vault`: Vault name or ID
- `item`: Item title or ID
- `section`: (Optional) Section name
- `field`: Field label (e.g., `password`, `username`, `credential`)

Query parameters for special attributes:

```bash
op read "op://vault/item/field?attribute=otp"              # one-time password
op read "op://vault/item/private key?ssh-format=openssh"   # SSH key format
```

## Resolving Secrets

There are three ways to resolve `op://` references, each suited to a different scenario:

### op read — Single secret

Read one secret value. Best for one-off lookups or piping into another command.

```bash
op read "op://vault/item/password"                            # stdout
op read --out-file=token.txt "op://vault/item/credential"     # file (no terminal output)

# Use in command substitution
docker login -u "$(op read op://prod/docker/username)" \
             -p "$(op read op://prod/docker/password)"
```

### op run — Environment variable injection

Scans env vars for `op://` references, resolves them, then runs a subprocess. Secrets are masked in output by default. Best for running apps or scripts that read config from env vars.

```bash
# From exported variables
export DB_PASSWORD="op://app-prod/db/password"
op run -- node app.js

# From .env file
op run --env-file="./prod.env" -- node app.js

# Unmasked (use with caution)
op run --no-masking -- printenv DB_PASSWORD
```

**Shell expansion gotcha:** The shell expands `$VAR` *before* `op run` sees it. Use a subshell to defer expansion:

```bash
# WRONG — shell expands $MY_VAR immediately
MY_VAR=op://vault/item/field op run --no-masking -- echo "$MY_VAR"

# CORRECT — subshell defers expansion
MY_VAR=op://vault/item/field op run --no-masking -- sh -c 'echo "$MY_VAR"'
```

### op inject — Template file injection

Replaces `op://` references in a template file with resolved values. Best for generating config files from templates.

```bash
op inject -i config.yml.tpl -o config.yml
```

Template (`config.yml.tpl`):
```yaml
database:
  host: http://localhost
  port: 5432
  username: op://prod/mysql/username
  password: op://prod/mysql/password
```

Delete resolved config files when no longer needed — they contain plaintext secrets.

### Environment Differentiation

Organize vaults by environment with the same item structure, then use a shell variable to switch contexts:

```bash
# .env file uses a variable for the vault name
DB_PASSWORD="op://$APP_ENV/mysql/password"

# Switch environment at runtime
APP_ENV=prod op run --env-file="./app.env" -- ./deploy.sh
APP_ENV=dev  op run --env-file="./app.env" -- ./deploy.sh

# Also works with op inject
APP_ENV=prod op inject -i config.yml.tpl -o config.yml
```

## Common Workflows

### Create an item (non-sensitive fields as flags, sensitive via template)

```bash
# Simple: non-sensitive fields only
op item create --category=Login --title="Netflix" --vault=Private \
  --url='https://www.netflix.com/login' \
  --generate-password='letters,digits,symbols,32' \
  'username=user@example.com'

# Secure: use a JSON template for sensitive values
op item template get --out-file=/tmp/login.json "Login"
# edit /tmp/login.json, then:
op item create --template=/tmp/login.json
rm /tmp/login.json
```

### Edit an item

```bash
op item edit <title-or-id> --title="New Title"
op item edit <title-or-id> 'username=new-user'
op item edit <title-or-id> 'Subscription.Renewal Date[date]=2025-12-31'
op item edit <title-or-id> 'Old Field[delete]'

# For sensitive value changes, use a template
op item get <title-or-id> --format json > /tmp/item.json
# edit /tmp/item.json, then:
op item edit <title-or-id> --template=/tmp/item.json
rm /tmp/item.json
```

### Search and filter

```bash
op item list --format=json                             # all items
op item list --vault=<vault> --format=json             # by vault
op item list --categories=Login --format=json          # by category
op item list --tags=production --format=json           # by tag
op item get <title-or-id> --format=json                # full details
op item get <title-or-id> --fields label=username,label=password --format=json
```

For the full command reference (all subcommands, flags, field types), see [reference/commands.md](reference/commands.md).
For naming conventions and vault organization guidelines, see [reference/conventions.md](reference/conventions.md).
