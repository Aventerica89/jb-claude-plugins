# 1Password Item Naming Conventions

## Item Name Rules (CRITICAL)

NEVER use `#`, `/`, spaces, or non-ASCII characters in item titles.
These characters break `op://` secret references silently.

**Pattern:** lowercase hyphenated slug from `package.json` `name`
**Examples:** `clarity`, `vaporforge`, `urls-to-go`, `wp-dispatch`

**Derive slug:**
1. Read `name` from `package.json`
2. Lowercase, replace any character not in `[a-z0-9]` with `-`, collapse consecutive `-` to single `-`
3. Trim leading/trailing `-`
4. Reject if empty or contains prohibited chars — abort setup

**One item per project:** all env vars as fields on a single item.

---

## Field Naming

Field names = exact env var names from source code (e.g. `GOOGLE_CLIENT_ID`).
Case-sensitive. Use SCREAMING_SNAKE_CASE.

---

## Reference Format

```
op://VaultName/item-slug/FIELD_NAME
```

Custom fields do NOT need a `/credential` suffix. Only the built-in `credential` field needs it.

---

## Template Syntax (Critical Distinction)

| Tool | Syntax | Writes to disk? |
|------|--------|----------------|
| `op inject -i tpl -o .env.local` | `{{ op://Vault/item/FIELD }}` | Yes |
| `op run --env-file=tpl -- cmd` | `op://Vault/item/FIELD` (bare) | No |

**Using `{{ }}` with `op run` causes SILENT failure** — vars come out empty, no error.

---

## Comment Caveat for op inject

`op inject` processes ALL `{{ }}` blocks including commented lines.
NEVER put `op://` references inside `#`-commented lines in `.tpl` files.

---

## Vault Convention

Always use **App Dev** vault for project secrets. Never Business vault.
