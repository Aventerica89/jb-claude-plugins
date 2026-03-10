---
name: auth-inject
description: This skill should be used when the user asks to "test auth injection", "verify credentials inject", "preflight check OAuth", "check if op run works", "validate dev auth before starting", "test that 1Password credentials resolve", "dry run auth injection", or wants to confirm that `op run` will successfully inject credentials before spinning up the dev server. Performs a non-destructive preflight validation of the local OAuth credential injection.
version: 0.1.0
---

# Auth Inject (Preflight Validation)

Validates that `op run` can successfully inject all credentials from `.env.local.dev.tpl` before the dev server starts. Catches expired sessions, missing items, and malformed `op://` paths without starting the server.

**No server is started. No credentials are written to disk.**

---

## Readiness Check

```bash
which op || echo "NOT INSTALLED"
op vault list 2>&1 | head -1
```

If `op` is not installed or not authenticated, stop: "1Password CLI not authenticated — run `op signin` first."

---

## Step 1: Find the Template

```bash
ls .env.local.dev.tpl 2>/dev/null || echo "NOT FOUND"
```

If missing: "No `.env.local.dev.tpl` found. Run `/setup-local-auth` first." and stop.

---

## Step 2: Run Preflight Injection

Execute `op run` with a no-op command to trigger credential resolution without starting anything:

```bash
op run --env-file=.env.local.dev.tpl -- env 2>&1
```

Filter the output to only show injected vars (vars that appear in the template), with values masked:

```bash
op run --env-file=.env.local.dev.tpl -- env 2>&1 | \
  grep -f <(grep -oE '^[A-Z_]+' .env.local.dev.tpl) | \
  sed 's/=.*/=***/'
```

---

## Step 3: Interpret Results

**Success:** All template vars appear in the filtered output with `***` masked values.

```
Injection OK

GOOGLE_CLIENT_ID=***
GOOGLE_CLIENT_SECRET=***
TODOIST_CLIENT_ID=***
TODOIST_CLIENT_SECRET=***

All 4 credentials resolved. Ready to run: npm run dev:auth
```

**Partial failure:** Some vars are missing from the output. This means those `op://` references failed to resolve. Report which vars are missing.

**Full failure / non-zero exit:** `op run` itself failed. Report the exact stderr output. Common causes:

| Error | Cause | Fix |
|-------|-------|-----|
| `[ERROR] 401` | Session expired | `op signin` |
| `[ERROR] item not found` | Item doesn't exist in vault | `/setup-local-auth` or `/auth-rotate` |
| `[ERROR] field not found` | Field missing on item | Add field in 1Password or `/auth-rotate` |
| `invalid secret reference` | Malformed `op://` path | Check `.env.local.dev.tpl` syntax |
| `op: command not found` | Not installed | Install from 1password.com |

---

## Step 4: Check `dev:auth` Script

```bash
grep '"dev:auth"' package.json
```

If the script exists, show it for confirmation. If it's missing, note it and suggest `/setup-local-auth`.

---

## Output Format

**All OK:**
```
Preflight: PASS

Resolved credentials (values masked):
  GOOGLE_CLIENT_ID       = ***
  GOOGLE_CLIENT_SECRET   = ***
  TODOIST_CLIENT_ID      = ***

Ready: npm run dev:auth
```

**With issues:**
```
Preflight: FAIL

Resolved:  2/3 credentials
Failed:    TODOIST_CLIENT_ID — item not found (clarity-dev-auth in App Dev)

Fix: run /auth-rotate to add TODOIST_CLIENT_ID, or /setup-local-auth to re-run setup
```
