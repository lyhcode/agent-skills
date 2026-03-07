---
name: 1password-cli
description: >
  Manage 1Password items, vaults, and secrets using the op CLI.
  Use when the user wants to create, read, update, or delete passwords and secrets,
  manage vaults, inject secrets into config files, run commands with secret environment variables,
  or read secret references. Triggers on mentions of 1Password, op CLI, password manager, or secret management.
---

# 1Password CLI (op) Assistant

Help users manage 1Password secrets and items using the `op` CLI.

## Prerequisites

- The `op` CLI must be installed (`brew install 1password-cli` on macOS)
- Desktop app integration must be enabled (1Password app > Settings > Developer > Integrate with 1Password CLI)
- Verify setup: `op whoami`

## Security Rules

**CRITICAL — follow these rules strictly:**

1. **NEVER output raw secret values** to the terminal unless the user explicitly asks. Use `op read --out-file` to pipe to files or `op run` to inject as env vars instead.
2. **Use `--format=json`** for all list/get commands so output is structured and parseable.
3. **Use JSON templates** (not inline assignment) when creating/editing items with sensitive values — command arguments can be visible to other processes on your machine.
4. **Prefer `op://` secret references** over reading secrets into variables.
5. **NEVER store or log** the output of secret values in files that might be committed to version control.
6. **Delete template files** after using them to create/edit items.

## Secret References (`op://`)

The secret reference format:

```
op://<vault>/<item>/[<section>/]<field>
```

- `vault`: Vault name or ID
- `item`: Item title or ID
- `section`: (Optional) Section name
- `field`: Field label (e.g., `password`, `username`, `credential`)

Query parameters for additional attributes:

```bash
# Get one-time password
op read "op://vault/item/field?attribute=otp"

# Get SSH key in OpenSSH format
op read "op://vault/item/private key?ssh-format=openssh"
```

Available field attributes: `type`, `value`, `title`, `id`, `purpose`, `otp`
Available file attributes: `content`, `size`, `id`, `name`, `type`

## Core Commands

### Authentication & Status

```bash
op whoami                          # Check current session
op account list                    # List configured accounts
op signin                          # Sign in (or switch account)
op signout                         # End session
op update                          # Check for updates
```

### Item Operations

```bash
# List items
op item list --format=json
op item list --vault=<vault> --format=json
op item list --categories=Login --format=json
op item list --tags=<tag> --format=json

# Get item details
op item get <title-or-id> --format=json
op item get <title-or-id> --fields label=username,label=password --format=json

# Create item with flags (non-sensitive fields only)
op item create --category=Login --title="Netflix" --vault=Private \
  --url='https://www.netflix.com/login' \
  --generate-password='letters,digits,symbols,32' \
  --tags=streaming,entertainment \
  'username=user@example.com'

# Create item with JSON template (recommended for sensitive values)
op item template get --out-file=/tmp/login.json "Login"
# Edit /tmp/login.json to add your information, then:
op item create --template=/tmp/login.json
# Delete the template file after use!

# Create from existing item
op item get "Netflix" --format json | op item create --title="Shared Netflix" -

# Edit item
op item edit <title-or-id> --title="New Title" --url=https://new-url.com
op item edit <title-or-id> 'username=new-user'
op item edit <title-or-id> 'Subscription.Renewal Date[date]=2025-12-31'

# Delete custom field (use [delete] as fieldType)
op item edit <title-or-id> 'Field Name[delete]'

# Edit with JSON template (for sensitive values)
op item get <title-or-id> --format json > /tmp/item.json
# Edit /tmp/item.json, then:
op item edit <title-or-id> --template=/tmp/item.json

# Delete item
op item delete <title-or-id>
op item delete <title-or-id> --archive  # Move to archive instead
```

### Assignment Statement Format

```
[<section>.]<field>[[<fieldType>]]=<value>
```

- For built-in fields: omit `fieldType`, use the field id from the template
- For custom fields: specify `fieldType` (defaults to `password` if omitted)
- Escape `.` `=` `\` in section/field names with backslash
- Field types: `text`, `password`, `otp`, `file`, `url`, `email`, `date`, `monthYear`
- Use `[delete]` to remove a custom field

### Special Field Types

```bash
# Add one-time password (OTP)
op item create --category=Login --title="My App" \
  'OTP[otp]=otpauth://totp/Example:user?secret=JBSWY3DPEHPK3PXP&issuer=Example'

# Attach a file
op item create --category=Login --title="My App" \
  'Receipt[file]=/path/to/receipt.pdf'
# Omit field name to preserve original filename:
  '[file]=/path/to/file.pdf'
```

### Vault Operations

```bash
op vault list --format=json
op vault get <name-or-id> --format=json
op vault create <name> --description="My vault"
op vault edit <name-or-id> --name="New Name"
op vault delete <name-or-id>

# Manage vault access
op vault user grant --vault=<vault> --user=<user> --permissions=allow_viewing,allow_editing
op vault user revoke --vault=<vault> --user=<user>
op vault user list <vault>
op vault group grant --vault=<vault> --group=<group> --permissions=allow_viewing
op vault group revoke --vault=<vault> --group=<group>
```

### Resolving Secrets: op read, op run, op inject

#### op read — Read a single secret

```bash
# Print to stdout
op read "op://vault/item/password"

# Save to file (avoids terminal output)
op read --out-file=token.txt "op://vault/item/credential"

# Use in scripts
docker login -u "$(op read op://prod/docker/username)" \
             -p "$(op read op://prod/docker/password)"
```

#### op run — Pass secrets as environment variables

`op run` scans environment variables for `op://` references, resolves them, then runs the command in a subprocess. Secrets are masked in output by default.

```bash
# Export variables with secret references, then run
export DB_USER="op://app-prod/db/user"
export DB_PASSWORD="op://app-prod/db/password"
op run -- node app.js

# Use with environment files
op run --env-file="./prod.env" -- node app.js

# Show unmasked values (use with caution)
op run --no-masking -- printenv DB_PASSWORD
```

**Shell expansion gotcha:** When you set a variable and use `op run` in the same command, the shell expands the variable *before* `op run` can substitute. Use a subshell:

```bash
# WRONG — shell expands $MY_VAR before op run
MY_VAR=op://vault/item/field op run --no-masking -- echo "$MY_VAR"

# CORRECT — expand in subshell
MY_VAR=op://vault/item/field op run --no-masking -- sh -c 'echo "$MY_VAR"'
```

#### op inject — Inject secrets into config files

Replace `op://` references in template files with actual secrets. References are used **directly** (no mustache/handlebars syntax).

```bash
op inject -i config.yml.tpl -o config.yml
```

Template example (`config.yml.tpl`):
```yaml
database:
  host: http://localhost
  port: 5432
  username: op://prod/mysql/username
  password: op://prod/mysql/password
```

**IMPORTANT:** Delete resolved config files when no longer needed.

### Environment Differentiation

Organize vaults by environment (dev/staging/prod) with the same item structure, then use shell variables to switch:

```bash
# In your .env file:
DB_PASSWORD="op://$APP_ENV/mysql/password"

# Switch environment:
APP_ENV=prod op run --env-file="./app.env" -- ./deploy.sh
APP_ENV=dev  op run --env-file="./app.env" -- ./deploy.sh
```

This also works with `op inject`:
```bash
APP_ENV=prod op inject -i config.yml.tpl -o config.yml
```

### Document Operations

```bash
op document list --format=json
op document get <title-or-id> --out-file=output.pdf
op document create <file-path> --title="My Document" --vault=<vault>
op document edit <title-or-id> <new-file-path>
op document delete <title-or-id>
```

### Generate Passwords

```bash
# Default 32-char password
op item create --category=Password --title="Generated" --generate-password

# Custom recipe
op item create --category=Password --title="Strong" \
  --generate-password='letters,digits,symbols,40'
```

### Shell Plugins

Authenticate third-party CLIs with biometrics. Credentials are stored in 1Password.

```bash
op plugin list                     # List available plugins
op plugin init <cli-name>          # Set up a plugin (e.g., aws, gh, stripe)
op plugin run <command>            # Run with provisioned credentials
op plugin inspect <cli-name>       # View configured credentials
op plugin clear <cli-name>         # Clear saved defaults
```

Supported CLIs include: AWS, GitHub, GitLab, Stripe, Terraform, Vercel, DigitalOcean, Heroku, and 60+ others.

### Service Accounts (Automation)

For CI/CD and automated scripts, use service accounts instead of personal accounts:

```bash
# Set service account token
export OP_SERVICE_ACCOUNT_TOKEN="<your-token>"

# Now op commands use the service account
op item list --format=json
op run --env-file=prod.env -- ./deploy.sh
```

Service accounts support restricting access to specific vaults (principle of least privilege). Create up to 100 per organization.

## Item Categories

Run `op item template list` for all available categories. Common ones:
- Login, Password, Identity, Credit Card, Secure Note
- API Credential, Database, Server, SSH Key
- Software License, Document

## Best Practices

1. **Keep `op` updated** — run `op update` regularly
2. **Least privilege** — use service accounts with dedicated, scoped vaults
3. **Use JSON templates** for items containing sensitive values
4. **Use `op run` or `op inject`** instead of reading secrets into shell variables
5. **Delete resolved config files** after use

For complete command reference, see [reference/commands.md](reference/commands.md).
For naming conventions, field types, vault organization, and best practices checklist, see [reference/conventions.md](reference/conventions.md).
