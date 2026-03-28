---
name: sac-setup
description: This skill should be used when the user asks to "set up a service account", "stop Touch ID prompts for op CLI", "add OP_SERVICE_ACCOUNT_TOKEN", "configure no-prompt 1Password access", or wants `op` CLI calls to work without biometric prompts. Sets up OP_SERVICE_ACCOUNT_TOKEN in ~/.zshrc.
version: 1.0.0
---

# SAC Setup (Service Account Token)

Sets up `OP_SERVICE_ACCOUNT_TOKEN` in `~/.zshrc` so direct `op` CLI calls (`op inject`, `op run`, `op read`, `op item`) work without Touch ID prompts.

**Does NOT affect:** shell plugins (vercel, stripe, etc.) — those use their own biometric auth flow and ignore this token.

---

## Step 1: Check if already configured

```bash
grep -q "OP_SERVICE_ACCOUNT_TOKEN" ~/.zshrc && echo "ALREADY SET" || echo "NOT SET"
```

If already set: show the line and ask if user wants to update it.

---

## Step 2: Create the service account (manual step)

The user must create the SA in 1Password web — this cannot be automated.

Give these exact instructions:
1. Go to `my.1password.com` → Integrations → Service Accounts
2. Click "New Service Account"
3. Name: `Local Dev` (or `Local Dev — <project>` for project-scoped)
4. Vault access: App Dev (Read)
5. Copy the `ops_eyJ...` token — it is shown only once

Ask: "Paste your service account token (starts with `ops_`):"

Validate the token starts with `ops_`. If it doesn't, reject and ask again.

---

## Step 3: Add to ~/.zshrc

Check for existing entry:
```bash
grep -n "OP_SERVICE_ACCOUNT_TOKEN" ~/.zshrc
```

If not present, use Edit to append to `~/.zshrc` — NEVER use Write (would overwrite the entire file):
```
# 1Password service account — no-prompt access for op CLI (read-only, App Dev vault)
export OP_SERVICE_ACCOUNT_TOKEN="<token>"
```

---

## Step 4: Verify in new shell context

```bash
env OP_SERVICE_ACCOUNT_TOKEN="<token>" op vault list 2>&1 | head -3
```

Expected: vault list returns without Touch ID prompt.

If it fails with auth error: the token may not have access to App Dev — ask user to check vault permissions in 1Password web.

---

## Completion

```
SAC setup complete

Token added to ~/.zshrc (OP_SERVICE_ACCOUNT_TOKEN)

Effective for: op read, op inject, op run, op item
NOT effective for: shell plugins (vercel, stripe — use their own biometric auth)

IMPORTANT: Open a new terminal for the token to take effect.
Existing terminal sessions (including this Claude Code session) will
still prompt Touch ID until restarted.
```
