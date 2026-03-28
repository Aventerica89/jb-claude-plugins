---
name: cicd-setup
description: This skill should be used when the user asks to "set up 1Password in CI", "add secrets to GitHub Actions", "wire load-secrets-action", "inject 1P secrets in GHA", or "configure 1Password for CI/CD". Generates a GitHub Actions step using 1password/load-secrets-action@v2 for the current project's secrets.
version: 1.0.0
---

# CI/CD Setup (GitHub Actions)

Generates a `load-secrets-action` step for GitHub Actions that injects the project's 1P secrets.

---

## Step 1: Get project item fields

Derive slug from `package.json` `name` (sanitize to lowercase-hyphenated).

```bash
op item get "<slug>" --vault "App Dev" --format json 2>/dev/null | jq -r '.fields[].label'
```

If item not found: "Run the `setup` skill first to create the 1P item, then come back to `cicd-setup`."

---

## Step 2: Ask about service account

Ask: "Do you have a dedicated 'GitHub Actions' service account in 1Password, or should I generate instructions for creating one?"

If new SA needed, give these instructions:
1. Go to `my.1password.com` → Integrations → Service Accounts
2. Name: `GitHub Actions — <repo-name>`
3. Vault access: App Dev (Read)
4. Copy the `ops_eyJ...` token

Then: "Store the token as a GitHub Actions secret named `OP_SERVICE_ACCOUNT_TOKEN`:"

```bash
echo "<token>" | gh secret set OP_SERVICE_ACCOUNT_TOKEN -R <owner>/<repo>
```

---

## Step 3: Generate workflow snippet

Output the complete step to insert in `.github/workflows/*.yml`:

```yaml
- name: Load secrets from 1Password
  uses: 1password/load-secrets-action@v2
  with:
    export-env: true
  env:
    OP_SERVICE_ACCOUNT_TOKEN: ${{ secrets.OP_SERVICE_ACCOUNT_TOKEN }}
    FIELD_NAME: op://App Dev/<slug>/FIELD_NAME
    FIELD_NAME_2: op://App Dev/<slug>/FIELD_NAME_2
    # ... one line per secret field from Step 1
```

Show where to insert in the workflow: before any step that uses the secrets, typically after `actions/checkout`.

---

## Notes

- `export-env: true` exports all resolved secrets as env vars for subsequent steps
- GHA automatically masks injected values as `***` in logs
- No `checkout` step is required before `load-secrets` — it can be the first step
- Shared credentials (Cloudflare, Stripe, etc.) live on their own 1P items — add manually if needed
- Skip PUBLIC vars (`NEXT_PUBLIC_*`, `VITE_*`) — those don't need 1P injection
