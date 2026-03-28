# 1Password CLI — Vault & Item Operations

> Source: https://developer.1password.com/llms-cli.txt (official 1Password CLI reference)

---

## Environment Variables

Full reference for all `op` environment variables:

| Variable | Description |
|----------|-------------|
| `OP_SERVICE_ACCOUNT_TOKEN` | Authenticate with a service account (no Touch ID). See sac-setup skill. |
| `OP_ACCOUNT` | Default account for all commands (sign-in address or ID). Useful with multiple accounts. |
| `OP_BIOMETRIC_UNLOCK_ENABLED` | Toggle app integration on/off (`true`/`false`). Useful to temporarily disable biometric. |
| `OP_CACHE` | Cache items/vault info between commands (`true`/`false`). Default: `true`. Set `false` if getting stale values. |
| `OP_FORMAT` | Global output format (`human-readable`/`json`). Default: `human-readable`. |
| `OP_DEBUG` | Debug mode (`true`/`false`). Default: `false`. |
| `OP_RUN_NO_MASKING` | Disable `***` masking in `op run` output. Use when debugging why secrets aren't resolving. |
| `OP_INCLUDE_ARCHIVE` | Allow retrieval of archived items (`true`/`false`). Default: `false`. |
| `OP_SESSION` | Session token from manual sign-in (legacy, prefer app integration). |
| `OP_CONFIG_DIR` | Override config directory (default: `~/.config/op/`). |
| `OP_CONNECT_HOST` | Connect server host URL (for team/self-hosted setups). |
| `OP_CONNECT_TOKEN` | Connect server token. |

---

## Authentication

### App integration (recommended — Touch ID)
```bash
op vault list    # triggers auth prompt on first use per terminal session
```

Auth expires after 10 minutes of inactivity, hard limit 12 hours. Authorization is per-terminal-session.

### Service account (no-prompt automation)
```bash
export OP_SERVICE_ACCOUNT_TOKEN="ops_eyJ..."
op vault list    # no Touch ID prompt
```

See `sac-setup` skill for full setup. Effective for `op read`, `op inject`, `op run`, `op item`. Does NOT affect shell plugins (vercel, stripe, etc. use their own biometric flow).

### Multiple accounts
```bash
op signin                          # interactive — select from accounts added to 1Password app
op vault list --account my.1password.com   # per-command account override
export OP_ACCOUNT=my.1password.com         # set default for session
op account use --account my.1password.com  # (Beta) set default for current terminal
```

---

## Vault Commands

### List vaults
```bash
op vault list
```
Returns: all vaults the current user has read access to (ID, NAME, TYPE, ITEMS, UPDATED).

### Create a vault
```bash
op vault create "App Dev"
```
Returns vault ID and name on success.

### Check if vault exists
```bash
op vault get "App Dev" 2>/dev/null && echo "EXISTS" || echo "NOT FOUND"
```

### Get vault details
```bash
op vault get "App Dev"
```

---

## Item Commands

### Create a new item (Login category with custom fields)

Using assignment statements (safe for non-sensitive values):
```bash
op item create \
  --category Login \
  --title "project-slug" \
  --vault "App Dev" \
  "FIELD_NAME=value" \
  "FIELD_NAME_2=value2"
```

For sensitive values — use JSON template to avoid shell history exposure:
```bash
# 1. Get template
op item template get Login > /tmp/item-template.json

# 2. Create item from template
op item create \
  --category Login \
  --title "project-slug" \
  --vault "App Dev" \
  --template /tmp/item-template.json

# 3. Clean up
rm /tmp/item-template.json
```

Then add secrets via `op item edit` (see below).

### Add/update fields on existing item
```bash
op item edit "project-slug" \
  --vault "App Dev" \
  "FIELD_NAME[text]=new_value" \
  "ANOTHER_FIELD[password]=secret_value"
```

Field type syntax: `"FIELD_NAME[type]=value"`
- `[text]` — plain text field
- `[password]` — masked field (recommended for secrets)
- `[concealed]` — alias for password

To delete a custom field:
```bash
op item edit "project-slug" --vault "App Dev" "FIELD_NAME[delete]"
```

### Get item details
```bash
op item get "project-slug" --vault "App Dev"
op item get "project-slug" --vault "App Dev" --format json
```

### List items in vault
```bash
op item list --vault "App Dev"
```

### Read a specific field value
```bash
op read "op://App Dev/project-slug/FIELD_NAME"
```

### Get the exact `op://` reference for a field (canonical — use this for templates)
```bash
# Single field
op item get "project-slug" --format json --fields FIELD_NAME | jq .reference
# → "op://App Dev/project-slug/FIELD_NAME"

# All fields on an item
op item get "project-slug" --vault "App Dev" --format json | jq '.fields[] | {label: .label, reference: .reference}'
```

Use `jq .reference` to get the exact canonical reference rather than constructing it manually — guarantees correct vault name, item name, and field name.

---

## Secret References

### Format
```
op://VaultName/ItemName/[SectionName/]FieldName
```

Rules:
- **VaultName**: exact vault name from `op vault list`
- **ItemName**: item title (no `#`, `/`, spaces, non-ASCII — see naming.md)
- **SectionName**: optional; only needed if field is inside a named section
- **FieldName**: exact field label (case-sensitive)

Custom fields do NOT need `/credential` suffix. Only the built-in `credential` field needs it.

### Template syntax — CRITICAL DISTINCTION

| Tool | Syntax | Writes to disk? |
|------|--------|----------------|
| `op inject -i tpl -o .env.local` | `{{ op://Vault/item/FIELD }}` | Yes |
| `op run --env-file=tpl -- cmd` | `op://Vault/item/FIELD` (bare) | No |

**Using `{{ }}` with `op run` causes SILENT failure** — vars come out empty, no error.

### op inject example (.env.local.tpl)
```bash
DATABASE_URL={{ op://App Dev/project-slug/DATABASE_URL }}
API_KEY={{ op://App Dev/project-slug/API_KEY }}
```

Run: `op inject -i .env.local.tpl -o .env.local`

### op run example (.env.local.dev.tpl)
```bash
DATABASE_URL=op://App Dev/project-slug/DATABASE_URL
API_KEY=op://App Dev/project-slug/API_KEY
```

Run: `op run --env-file=.env.local.dev.tpl -- npm run dev`

### Multi-environment template variables (advanced)
Use `{{ }}` placeholders within the item name to load different secrets per environment:
```bash
# .env.tpl
DATABASE_URL={{ op://App Dev/project-slug-{{ENV}}/DATABASE_URL }}
```

```bash
ENV=staging op inject -i .env.tpl -o .env.local
# resolves: op://App Dev/project-slug-staging/DATABASE_URL
```

Requires separate 1Password items per environment (`project-slug-dev`, `project-slug-staging`, `project-slug-prod`).

---

## Debugging

### OP_DEBUG — verbose output
```bash
OP_DEBUG=true op item get "project-slug" --vault "App Dev"
```

### OP_RUN_NO_MASKING — see actual resolved values
```bash
OP_RUN_NO_MASKING=true op run --env-file=.env.local.dev.tpl -- env | grep MY_VAR
```
Use this when `op run` resolves `***` and you can't tell if the value is correct or empty.

### OP_CACHE=false — force fresh reads
```bash
OP_CACHE=false op read "op://App Dev/project-slug/FIELD_NAME"
```
Use when getting stale/outdated values after editing an item in the 1Password app.

### Verify auth state
```bash
op whoami           # shows current account, user, auth method
op vault list       # quick auth check — fails if not authenticated
```

---

## Common Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `[ERROR] 401` | Not authenticated | Run `op signin` or set `OP_SERVICE_ACCOUNT_TOKEN` |
| `[ERROR] 403` | No vault access | Check vault permissions for current account/SA |
| `invalid secret reference` | Malformed `op://` path | Use `op item get --format json \| jq '.fields[] .reference'` to get canonical path |
| Silent empty vars with `op run` | Used `{{ }}` syntax instead of bare `op://` | Use bare `op://` format in `--env-file` templates |
| Silent empty vars with `op inject` | Used bare `op://` instead of `{{ }}` | Use `{{ op://... }}` in `.tpl` files for `op inject` |
| `item not found` | Wrong item title or vault | Run `op item list --vault "App Dev"` to confirm exact title |
| Stale/wrong value returned | CLI cache has old data | Set `OP_CACHE=false` or pass `--cache=false` |
| Touch ID prompt every command | App integration not configured | Check 1Password app Settings → Developer → Integrate with CLI |
