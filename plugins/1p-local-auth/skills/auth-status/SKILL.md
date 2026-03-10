---
name: auth-status
description: This skill should be used when the user asks to "check auth status", "check OAuth credential status", "verify my local auth setup", "are my dev credentials configured", "show auth config", "which providers are set up", "check if 1Password items exist for auth", or wants to diagnose their local OAuth setup. Provides a read-only diagnostic of the 1Password-backed local dev auth setup.
version: 0.1.0
---

# Auth Status

Read-only diagnostic for the 1p-local-auth setup. Checks that all credentials referenced in `.env.local.dev.tpl` exist in 1Password, flags any override conflicts in `.env.local`, and reports the overall health of the local OAuth configuration.

**No changes are made.** This is a pure diagnostic.

---

## Readiness Check

Before reading 1Password, verify the CLI is available:

```bash
which op || echo "NOT INSTALLED"
op vault list 2>&1 | head -1
```

If `op` is not installed or the session is expired, report the specific error and stop: "1Password CLI not authenticated — run `op signin` first."

---

## Step 1: Find the Template

```bash
ls .env.local.dev.tpl 2>/dev/null || echo "NOT FOUND"
```

If `.env.local.dev.tpl` does not exist: report "No `.env.local.dev.tpl` found. Run `/setup-local-auth` to configure." and stop.

---

## Step 2: Parse Template References

Extract all `op://` references from the template:

```bash
grep -oP 'op://[^}]+' .env.local.dev.tpl
```

For each reference, parse out: vault, item name, field name.

Example parsing `op://App Dev/clarity-dev-auth/GOOGLE_CLIENT_ID`:
- Vault: `App Dev`
- Item: `clarity-dev-auth`
- Field: `GOOGLE_CLIENT_ID`

Also extract the env var name that the reference is assigned to (the left side of the `=`).

---

## Step 3: Check Each Credential in 1Password

For each parsed reference, verify the item and field exist and are non-empty using the 1Password MCP:

- Item exists in vault → **found** / **MISSING ITEM**
- Field exists on item → **found** / **MISSING FIELD**
- Field has a non-empty value → **has value** / **EMPTY**

Group results by provider where possible (infer from env var name prefix: `GOOGLE_`, `GITHUB_`, `TODOIST_`, etc.).

---

## Step 4: Check `.env.local` for Conflicts

```bash
# Get all var names from template
grep -oE '^[A-Z_]+' .env.local.dev.tpl 2>/dev/null
```

For each var name, check whether it also appears in `.env.local`:

```bash
grep -E "^VAR_NAME=" .env.local 2>/dev/null
```

Flag any matches as override conflicts — values in `.env.local` take precedence over `op run` injection.

---

## Step 5: Check `dev:auth` Script

```bash
grep '"dev:auth"' package.json
```

Report whether the script exists. If missing, note that `/setup-local-auth` can add it.

---

## Output Format

Present a clean diagnostic table:

```
Auth Status — clarity

Provider    Variable               1Password
Google      GOOGLE_CLIENT_ID       found
Google      GOOGLE_CLIENT_SECRET   found
Todoist     TODOIST_CLIENT_ID      MISSING ITEM
Todoist     TODOIST_CLIENT_SECRET  MISSING ITEM

Warnings:
  - GOOGLE_CLIENT_ID also in .env.local — .env.local value takes precedence
  - dev:auth script missing from package.json

Summary: 2/4 credentials OK, 2 missing, 1 conflict warning
```

Status values:
- `found` — credential resolved
- `MISSING ITEM` — 1Password item does not exist
- `MISSING FIELD` — item exists but field is missing
- `EMPTY` — field exists but has no value

---

## Remediation Hints

After the table, include brief remediation hints for any issues found:

- **MISSING ITEM / MISSING FIELD:** "Run `/setup-local-auth` to re-create credentials, or use `/auth-rotate` to add the missing field."
- **EMPTY value:** "Open 1Password and add a value to the `{field}` field in `{item}`."
- **`.env.local` conflict:** "Remove `{var}` from `.env.local` to let `op run` injection take effect, or leave it to keep the override intentional."
- **Missing `dev:auth` script:** "Run `/setup-local-auth` to add it, or add manually: `\"dev:auth\": \"op run --env-file=.env.local.dev.tpl -- npm run dev\"`."
