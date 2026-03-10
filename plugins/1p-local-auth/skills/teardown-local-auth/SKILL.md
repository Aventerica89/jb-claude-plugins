---
name: teardown-local-auth
description: This skill should be used when the user asks to "remove local auth setup", "teardown local OAuth", "remove dev auth", "clean up my local OAuth setup","undo local auth setup", "remove the dev:auth script", "delete the auth template", or wants to reverse a previous /setup-local-auth run. Cleanly removes all 1p-local-auth artifacts from the project without deleting the 1Password item.
version: 0.1.0
---

# Teardown Local Auth

Reverse of `/setup-local-auth`. Removes local artifacts (template file, `dev:auth` script, `.gitignore` entry) from the project. Does **not** delete the 1Password item by default — credentials are preserved for potential re-setup.

---

## Step 1: Inventory What Exists

Before making changes, audit what the setup created:

```bash
ls .env.local.dev.tpl 2>/dev/null && echo "EXISTS: .env.local.dev.tpl" || echo "NOT FOUND: .env.local.dev.tpl"
grep '"dev:auth"' package.json && echo "EXISTS: dev:auth script" || echo "NOT FOUND: dev:auth script"
grep -F '!.env*.tpl' .gitignore 2>/dev/null && echo "EXISTS: .gitignore entry" || echo "NOT FOUND: .gitignore entry"
```

Also note the 1Password item name (from the template if it exists):

```bash
grep -oP 'op://[^/]+/[^/]+' .env.local.dev.tpl 2>/dev/null | head -1
# Example: op://App Dev/clarity-dev-auth
```

Present the teardown plan and confirm with user before proceeding.

---

## Step 2: Remove `.env.local.dev.tpl`

```bash
rm .env.local.dev.tpl
```

Verify deletion:
```bash
ls .env.local.dev.tpl 2>/dev/null && echo "STILL EXISTS" || echo "Removed"
```

---

## Step 3: Remove `dev:auth` Script from `package.json`

Read `package.json`, find the `dev:auth` line, and remove it. Use the Edit tool — do not rewrite the entire file.

Ensure the JSON remains valid after removal (no trailing commas).

---

## Step 4: Remove `.gitignore` Entry (Conditional)

Check whether any other `.tpl` files remain in the project:

```bash
fd -e tpl
```

If `.env.local.tpl` or other template files still exist: **do not remove the `.gitignore` entry** — other templates still need it.

If no `.tpl` files remain: remove the `!.env*.tpl` line from `.gitignore`.

---

## Step 5: 1Password Item — Preserve, Don't Delete

Do **not** automatically delete the `{slug}-dev-auth` item from 1Password.

Instead, provide the exact command for manual deletion if the user wants it:

```
The 1Password item was NOT deleted automatically.

To delete it manually:
  op item delete clarity-dev-auth --vault "App Dev"

Recommendation: archive instead of delete — keeps credentials if you re-setup later:
  1Password app → find "clarity-dev-auth" → Move to Archive
```

Ask: "Would you like me to delete the 1Password item now, or leave it?" Only delete if explicitly confirmed.

---

## Completion Summary

```
Teardown complete

Removed:   .env.local.dev.tpl
Removed:   dev:auth script from package.json
Skipped:   .gitignore entry (.env.local.tpl still present)
Preserved: 1Password item "clarity-dev-auth" in App Dev

To delete the 1Password item:
  op item delete clarity-dev-auth --vault "App Dev"

To re-run setup in future:
  /setup-local-auth
```

---

## Partial Teardown

If the user only wants to remove specific artifacts (e.g., keep the 1Password item and template but remove the `dev:auth` script), handle only the requested subset. Confirm which parts to remove before acting.
