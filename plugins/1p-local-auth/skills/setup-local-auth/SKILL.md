---
name: setup-local-auth
description: This skill should be used when the user asks to "set up local auth", "configure local OAuth", "set up dev OAuth credentials", "set up 1Password for local auth", "configure Google auth locally", "set up GitHub OAuth for local dev", "add dev auth to this project", or mentions needing local OAuth credentials without swapping env files. Sets up a 1Password-backed local dev OAuth credential workflow using op run.
version: 0.1.0
---

# Setup Local Auth

Sets up a complete 1Password-backed OAuth credential workflow for local development. After setup, `npm run dev:auth` injects real OAuth credentials from 1Password at dev server start — no swapping `.env.local` files, no production credentials at risk.

**Prerequisite:** 1Password CLI (`op`) must be installed and authenticated.

---

## Readiness Check (Run First)

Before any other step, verify 1Password is available:

```bash
# Step 1: verify op is installed
which op || echo "NOT INSTALLED"

# Step 2: verify active session and vault access
op vault list
```

If `op` is not installed: stop and instruct the user to install it from `1password.com/downloads/mac/`.

If vault list fails: stop with the message "1Password CLI not authenticated — run `op signin` first."

---

## Step 1: Vault Selection

Run `op vault list` and display all available vaults. Ask the user which vault to use, defaulting to `App Dev` if present. Store the selected vault name — it will appear in every `op://` reference.

---

## Step 2: Pre-Flight Conflict Check

Before making any changes, check what already exists and summarize:

```bash
# Check for existing files
ls -la .env.local.dev.tpl 2>/dev/null
grep '"dev:auth"' package.json
```

Present a clear before-you-proceed summary:
- `.env.local.dev.tpl` — exists / will create
- `dev:auth` script in `package.json` — exists / will add
- `.gitignore` `!.env*.tpl` entry — exists / will add

Confirm with user before making changes if conflicts exist.

---

## Step 3: Detect Auth Framework

Scan common paths for auth configuration:

```bash
for f in auth.ts src/auth.ts src/lib/auth.ts lib/auth.ts; do
  [ -f "$f" ] && echo "FOUND: $f" && break
done
```

Read the found file to identify active providers. If no file is found, ask the user which providers to configure (Google, GitHub, Todoist, other).

**Supported frameworks:** Better Auth, NextAuth/Auth.js v5.

For provider-specific env var names by framework, see `references/providers.md`.

---

## Step 4: Determine the Project Slug

Read `name` from `package.json`. Sanitize to only `[a-z0-9-]` characters — this becomes the 1Password item name `{slug}-dev-auth`.

Examples: `clarity` → `clarity-dev-auth`, `my-app` → `my-app-dev-auth`.

Reject and abort if the resulting slug is empty or contains prohibited characters (`#`, `?`, `&`, `/`, `\`, spaces, non-ASCII).

---

## Step 5: Create 1Password Item

Use the 1Password MCP to create one item per project with one field per OAuth credential:

- **Vault:** (user-selected from Step 1)
- **Item name:** `{slug}-dev-auth`
- **Fields:** one per env var (e.g., `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`)

Before creating: check if `{slug}-dev-auth` already exists in the vault. If it does, offer to update existing fields or abort.

For instructions on creating OAuth dev apps per provider, see `references/providers.md`.

---

## Step 6: Write `.env.local.dev.tpl`

Create the template file with `op://` references. Use the vault name from Step 1 and the item name from Step 4:

```bash
# Example output for a Google + Todoist setup
GOOGLE_CLIENT_ID=op://App Dev/clarity-dev-auth/GOOGLE_CLIENT_ID
GOOGLE_CLIENT_SECRET=op://App Dev/clarity-dev-auth/GOOGLE_CLIENT_SECRET
TODOIST_CLIENT_ID=op://App Dev/clarity-dev-auth/TODOIST_CLIENT_ID
TODOIST_CLIENT_SECRET=op://App Dev/clarity-dev-auth/TODOIST_CLIENT_SECRET
```

**Critical:** `op run --env-file` requires bare `op://vault/item/field` references — no braces. The `{{ op://... }}` double-brace format is for `op inject` only and will silently fail with `op run`.

---

## Step 7: Add `dev:auth` Script to `package.json`

Add to the `scripts` section:

```json
"dev:auth": "op run --env-file=.env.local.dev.tpl -- npm run dev"
```

If `dev` script uses a different command (e.g., `next dev --turbopack`), use that directly. Read the existing `dev` script and mirror it:

```json
"dev:auth": "op run --env-file=.env.local.dev.tpl -- {existing dev command}"
```

---

## Step 8: Patch `.gitignore`

Check for `.env*.tpl` allowance. If not present, add it:

```
!.env*.tpl
```

Ensure `.env.local` itself remains ignored.

---

## Step 9: Precedence Warning

Check whether any OAuth env vars from the new template also exist in `.env.local`:

```bash
# Extract var names from template
grep -oE '^[A-Z_]+' .env.local.dev.tpl | while read var; do
  grep -q "^$var=" .env.local 2>/dev/null && echo "CONFLICT: $var also in .env.local"
done
```

If conflicts found: warn the user. Values in `.env.local` take precedence over `op run` injection when both are present. Advise removing the conflicting vars from `.env.local`, or acknowledge the override is intentional.

---

## Step 10: Verification Run

Run a preflight check to confirm credentials resolve:

```bash
op run --env-file=.env.local.dev.tpl -- env | grep -E 'GOOGLE_|GITHUB_|TODOIST_' | sed 's/=.*/=***MASKED***/'
```

If this fails: diagnose the error. Common causes: expired `op` session, item not created, malformed `op://` path. Show the exact error and suggest the fix.

---

## Completion Summary

Present a clear completion report:

```
1p-local-auth setup complete

Created:  .env.local.dev.tpl (4 vars)
Updated:  package.json — added dev:auth script
Updated:  .gitignore — added !.env*.tpl
1Password: clarity-dev-auth created in App Dev vault

Start local dev with OAuth:
  npm run dev:auth

Check credential status:
  /auth-status

Rotate a credential:
  /auth-rotate
```

---

## Additional Resources

### Reference Files

- **`references/providers.md`** — Provider-specific OAuth dev app setup (Google, GitHub, Todoist) and env var names per auth framework

---

## Edge Cases

**Monorepo / no package.json at root:** Ask the user which `package.json` to update.

**Provider not in supported list:** Collect the env var names manually from the user. Ask: "What environment variable names does your auth config use for this provider?"

**`op run` vs `op inject`:** `op inject` renders a template file (`{{ }}` syntax) to a new file. `op run` injects secrets directly into a process's environment. This plugin uses `op run` at dev server start — the template uses `{{ }}` syntax purely for `op inject` compatibility if the user also wants to generate a static `.env.local.dev` file.
