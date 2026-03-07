# 1Password CLI Conventions

Best practices for storing credentials in 1Password with `op` CLI integration.

## Item Categories

Choose the right category for each type of credential:

| Category | CLI value | Use for |
|----------|-----------|---------|
| Login | `login` | Website accounts (username + password) |
| API Credential | `"API Credential"` | API tokens, PATs, service keys |
| SSH Key | `"SSH Key"` | SSH keys (integrates with SSH agent) |
| Server | `server` | Server access (URL + username + password) |
| Database | `database` | Database connections |
| Password | `password` | Standalone passwords |
| Secure Note | `"Secure Note"` | Free-form text notes |

### Built-in fields per category

| Category | Built-in fields |
|----------|----------------|
| Login | `username`, `password` |
| API Credential | `username`, `credential`, `expires`, `valid from` |
| Server | `url`, `username`, `password` |
| Database | `server`, `port`, `database`, `username`, `password`, `type` |
| SSH Key | `private_key`, `public_key` |

## Naming Conventions

### Human-facing items (Login, Server, Database)
Title case with spaces: `ServiceName - account/purpose`

```
Gandi - production
AWS - staging
GitHub - personal
PostgreSQL - app-db-prod
```

### Machine-readable items (API Credential, SSH Key)
All lowercase with dashes: `service-account-purpose`

```
gandi-production-pat
aws-staging-key
github-ci-token
deploy-prod-ssh-key
```

### Rules
- **No `/` in titles** — `op read` URI format `op://vault/item/field` conflicts with `/` in titles
- **No special characters** — stick to alphanumeric, dashes, spaces
- **Include purpose** — `gandi-prod-pat` not just `gandi-pat`, in case of multiple tokens
- **Use IDs for stability** — item IDs are more stable than names; names can be renamed

## Separate Accounts from Credentials

One account can have multiple API tokens with different permissions and expiry dates.

```
Gandi - production        (Login)     → account credentials
gandi-production-pat      (API Cred)  → full access token, expires 2027-03
gandi-production-readonly (API Cred)  → read-only token, expires 2026-12
```

API Credential items should ONLY contain the credential. Don't duplicate username/email/password.

```bash
# WRONG — mixing account info into PAT item
op item create --category "API Credential" \
  --title "gandi-prod-pat" \
  'credential=token-value' \
  'username=myuser' \
  'email=me@example.com'

# CORRECT — PAT item only has the credential
op item create --category "API Credential" \
  --title "gandi-prod-pat" \
  'credential=token-value' \
  'expires=2027-03-08'
```

## Field Types

Custom fields default to **concealed** (password-type). Use type annotations to control this.

### Type annotation syntax

```
'fieldname=value'                → concealed (default, for secrets)
'fieldname[text]=value'          → plain text (for non-sensitive data)
'fieldname[password]=value'      → concealed (explicit)
'fieldname[otp]=otpauth://...'   → one-time password
'fieldname[date]=2027-03-08'     → date
'fieldname[monthyear]=2027-03'   → month/year
'fieldname[url]=https://...'     → URL
'fieldname[email]=user@test.com' → email
'fieldname[phone]=+1234567890'   → phone
```

### Sections

Group related custom fields into sections:

```bash
op item create --category server \
  --title "App Server - prod" \
  'Server Config.host[text]=app.example.com' \
  'Server Config.port[text]=8080' \
  'Monitoring.endpoint[url]=https://monitor.example.com' \
  'Monitoring.api_key=secret-key'
```

Escape special characters in section/field names with `\`:
```bash
'My\.Section.field\=name[text]=value'
```

### Common mistakes

```bash
# WRONG — email stored as concealed
op item create --category login 'email=user@example.com'

# CORRECT — email stored as plain text
op item create --category login 'email[text]=user@example.com'

# WRONG — URL stored as concealed
op item create --category server 'endpoint=https://api.example.com'

# CORRECT — URL stored as URL type
op item create --category server 'endpoint[url]=https://api.example.com'
```

## Vault Organization

### Principles
- **Least privilege** — only grant access to vaults people/services need
- **Start simple** — one vault per department or project
- **Separate environments** — don't mix prod and dev secrets

### Recommended structure

| Vault | Purpose | Access |
|-------|---------|--------|
| Private/Employee | Personal work credentials | Individual only |
| Shared | Company-wide resources (Wi-Fi, office) | All team members |
| Engineering | Dev tools, CI/CD tokens | Engineering team |
| Production | Prod API keys, DB credentials | Senior engineers + service accounts |
| Staging | Staging environment secrets | Engineering team |

### Service Accounts
- Use dedicated service accounts for CI/CD with access scoped to specific vaults
- Don't use personal accounts in automated pipelines
- Rotate service account tokens regularly

## Tags

Use tags for cross-cutting concerns that span multiple vaults/categories:

```bash
op item create --category "API Credential" \
  --title "gandi-prod-pat" \
  --tags "production,api,gandi" \
  'credential=token-value'
```

- Tags can be nested with `/`: `cloud/aws`, `cloud/gcp`
- An item can have multiple tags
- Use for filtering: `op item list --tags production`
- Good tag examples: environment (`production`, `staging`), service (`gandi`, `aws`), team (`frontend`, `backend`)

## Complete Example

```bash
# 1. Create Login (account for humans)
op item create --category login \
  --title "Gandi - production" \
  --url "https://admin.gandi.net" \
  --tags "gandi,production" \
  'username=myuser' \
  'password=mypassword' \
  'email[text]=me@example.com'

# 2. Create API Credential (token for machines)
op item create --category "API Credential" \
  --title "gandi-production-pat" \
  --tags "gandi,production,api" \
  'credential=my-pat-token' \
  'expires=2027-03-08'

# 3. Use with CLI
gandi --token $(op read "op://Private/gandi-production-pat/credential") domain list

# 4. Or use op run with .env
echo 'GANDI_PAT=op://Private/gandi-production-pat/credential' > .env
op run --env-file=.env -- gandi domain list
```

## Checklist

- [ ] Correct category chosen (Login vs API Credential vs Server vs Database)
- [ ] Login and API Credential are separate items
- [ ] Machine-readable items are all lowercase with dashes
- [ ] No `/` or special characters in item titles
- [ ] Non-sensitive custom fields use `[text]`, `[url]`, or `[email]` type annotations
- [ ] API Credential only contains `credential` (no duplicated account info)
- [ ] Expiry date is set on tokens/PATs
- [ ] Tags added for filtering (`environment`, `service`)
- [ ] Secrets in `.env` files use `op://` references, not plaintext
- [ ] Service accounts (not personal) used for CI/CD
