# 1Password CLI — Vault & Item Operations

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

# 2. Create item from template (pipe to avoid disk exposure)
op item create \
  --category Login \
  --title "project-slug" \
  --vault "App Dev"
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

---

## Secret References

### Format
```
op://VaultName/ItemName/FieldName
```

Rules:
- **VaultName**: exact vault name from `op vault list`
- **ItemName**: item title (no `#`, `/`, spaces, non-ASCII — see naming.md)
- **FieldName**: exact field label (case-sensitive)

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

---

## Common Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `[ERROR] 401` | Not authenticated | Run `op signin` |
| `[ERROR] 403` | No vault access | Check vault permissions |
| `invalid secret reference` | Malformed `op://` path | Check vault/item/field names exactly |
| Silent empty vars with `op run` | Used `{{ }}` syntax instead of bare `op://` | Use bare `op://` format in `--env-file` templates |
| `item not found` | Wrong item title or vault | Run `op item list --vault "App Dev"` to confirm exact title |
