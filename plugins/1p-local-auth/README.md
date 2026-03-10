# 1p-local-auth

Manage local dev OAuth credentials via 1Password. Test any auth provider on any branch without swapping `.env.local` files or risking production credentials.

---

## The Problem

OAuth providers require pre-registered redirect URIs. Production credentials point to `https://yourapp.com`. Testing locally requires credentials registered against `http://localhost:3000` — but most teams handle this by:

- Manually swapping `.env.local` files
- Sharing dev credentials in Slack
- Using production credentials locally (risky)

This plugin sets up a clean workflow: one command starts your dev server with real OAuth credentials injected directly from 1Password.

---

## Prerequisites

- [1Password CLI (`op`)](https://developer.1password.com/docs/cli/get-started/) installed and authenticated
- 1Password account with a vault for dev credentials (e.g., `App Dev`)
- Project using Better Auth or NextAuth/Auth.js v5

---

## Installation

### Step 1: Add the marketplace

```bash
claude plugin marketplace add Aventerica89/jb-claude-plugins
```

### Step 2: Install the plugin

```bash
claude plugin install 1p-local-auth@jb-claude-plugins
```

### Step 3: Restart Claude Code

Restart your Claude Code session for the new skills to appear.

### Troubleshooting: SSH errors during install

If `claude plugin install` fails with `Permission denied (publickey)`, set a git URL rewrite to use HTTPS:

```bash
git config --global url."https://github.com/".insteadOf git@github.com:
```

Then retry the install command.

---

## Skills

### `/setup-local-auth`

Full interactive setup wizard. Run once per project.

1. Verifies 1Password is installed and authenticated
2. Lets you select the target vault
3. Detects your auth framework and active providers
4. Creates a `{project}-dev-auth` item in 1Password
5. Writes `.env.local.dev.tpl` with `op://` references
6. Adds `dev:auth` script to `package.json`
7. Patches `.gitignore`

**After setup:**
```bash
npm run dev:auth  # starts dev server with OAuth credentials injected
```

---

### `/auth-status`

Read-only diagnostic. Checks that all credentials referenced in `.env.local.dev.tpl` exist in 1Password and flags any override conflicts.

```
Provider    Variable               1Password
Google      GOOGLE_CLIENT_ID       found
Google      GOOGLE_CLIENT_SECRET   found
Todoist     TODOIST_CLIENT_ID      MISSING ITEM
```

---

### `/auth-inject`

Preflight validation. Runs `op run` with a no-op to confirm credentials resolve before starting the dev server. Catches expired sessions, missing items, and malformed paths.

```
Preflight: PASS

Resolved credentials (values masked):
  GOOGLE_CLIENT_ID       = ***
  GOOGLE_CLIENT_SECRET   = ***
```

---

### `/auth-rotate`

Targeted credential rotation. Updates a single field in the 1Password item — no template changes needed since templates reference field names, not values.

---

### `/teardown-local-auth`

Reverses setup. Removes `.env.local.dev.tpl`, the `dev:auth` script, and optionally the `.gitignore` entry. Does not delete the 1Password item by default (preserves credentials for re-setup).

---

## 1Password Storage Convention

One item per project, multiple fields:

```
Vault:      App Dev
Item name:  {project}-dev-auth
Fields:     GOOGLE_CLIENT_ID, GOOGLE_CLIENT_SECRET, ...
```

Template reference format:
```
op://App Dev/clarity-dev-auth/GOOGLE_CLIENT_ID
```

**Why no `#` characters?** The `op` CLI parses secret references as URLs. The `#` character is a URL fragment delimiter that silently breaks the parser. All item names use only `[a-z0-9-]` characters.

---

## Workflow

```
Setup (once):          /setup-local-auth
Daily dev:             npm run dev:auth
Check status:          /auth-status
Verify before start:   /auth-inject
Rotate credential:     /auth-rotate
Clean up:              /teardown-local-auth
```

---

## Supported Auth Frameworks

- **Better Auth** — reads `GOOGLE_CLIENT_ID`, `GITHUB_CLIENT_ID`, etc.
- **NextAuth / Auth.js v5** — reads `AUTH_GOOGLE_ID`, `AUTH_GITHUB_ID`, etc.

## Supported Providers (v1)

- Google
- GitHub
- Todoist

Other providers can be added manually during setup.

---

## License

MIT — JBMD Creations
