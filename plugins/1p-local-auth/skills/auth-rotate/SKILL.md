---
name: auth-rotate
description: This skill should be used when the user asks to "rotate a credential", "update OAuth secret", "change Google client secret", "update my dev auth credentials", "replace a 1Password credential", "my OAuth secret changed", "update local dev credentials", "add a missing credential", or needs to update a specific OAuth credential in the 1Password dev auth item. Performs targeted rotation of a single credential in the local dev auth setup.
version: 0.1.0
---

# Auth Rotate

Updates a single credential in the 1Password dev auth item. No template files change — templates reference field names, not values. After rotation, the next `npm run dev:auth` automatically uses the new value.

---

## Readiness Check

```bash
which op || echo "NOT INSTALLED"
op vault list 2>&1 | head -1
```

If `op` is not installed or not authenticated, stop: "1Password CLI not authenticated — run `op signin` first."

---

## Step 1: Identify What to Rotate

Ask the user: "Which credential needs to be updated?" If already specified in the request, confirm the understanding.

Determine:
1. **Variable name** — e.g., `GOOGLE_CLIENT_SECRET`
2. **New value** — the replacement credential (ask securely; do not log in output)
3. **Item name** — read from `.env.local.dev.tpl` to find the item that holds this field

```bash
# Replace GOOGLE_CLIENT_SECRET with the actual var name from user's request
VAR=GOOGLE_CLIENT_SECRET
grep "^${VAR}=" .env.local.dev.tpl
# Example output: GOOGLE_CLIENT_SECRET=op://App Dev/clarity-dev-auth/GOOGLE_CLIENT_SECRET
```

Parse out: vault (`App Dev`), item (`clarity-dev-auth`), field (`GOOGLE_CLIENT_SECRET`).

---

## Step 2: Confirm Before Writing

Present a clear confirmation before updating:

```
About to update:
  Vault:  App Dev
  Item:   clarity-dev-auth
  Field:  GOOGLE_CLIENT_SECRET
  Value:  [new value will replace existing]

Proceed? [y/N]
```

Wait for user confirmation.

---

## Step 3: Update via 1Password MCP

Use the 1Password MCP to update the field value on the existing item. Do not recreate the item — update only the specified field.

If the field does not exist on the item (new credential being added), create the field.

---

## Step 4: Verify Template Reference

After updating, confirm the template still references the correct item and field:

```bash
grep "VARIABLE_NAME" .env.local.dev.tpl
```

If the reference is missing or malformed, report it and offer to add the correct line.

---

## Step 5: Run Injection Verification

Run a quick preflight to confirm the new value resolves:

```bash
op run --env-file=.env.local.dev.tpl -- env 2>&1 | grep "^${VAR}=" | sed 's/=.*/=***/'
```

If the var appears in output: rotation successful.
If it fails: diagnose and report.

---

## Completion Output

```
Credential rotated

  Vault:  App Dev
  Item:   clarity-dev-auth
  Field:  GOOGLE_CLIENT_SECRET
  Status: updated, verified resolving

Restart dev server to pick up new value:
  npm run dev:auth
```

---

## Adding a New Provider

If the user is adding a credential for a new provider (not rotating an existing one):

1. Add the new field(s) to `{slug}-dev-auth` in 1Password via MCP
2. Add the corresponding `op://` reference lines to `.env.local.dev.tpl`
3. Run the inject verification (Step 5)
4. Check for any framework-specific env var names — see `references/providers.md` in the `setup-local-auth` skill

---

## Edge Cases

**Multiple credentials to rotate at once:** Handle one at a time, repeating Steps 1–5 for each. Batch operations risk partial failures that are harder to debug.

**Credential value contains special characters:** 1Password stores values verbatim — no escaping needed. The issue only arises in shell context, not in 1Password storage.

**Item not found:** If `{slug}-dev-auth` doesn't exist, the setup was never completed or the item was deleted. Run `/setup-local-auth` to re-create from scratch.
