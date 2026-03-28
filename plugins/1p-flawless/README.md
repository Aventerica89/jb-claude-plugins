# 1p-flawless

**Never type a password into your code again.**

Every app needs secrets: database passwords, API keys, login credentials for Google/GitHub/Stripe. The old way: paste them into a `.env` file, hope you don't accidentally commit it to GitHub, manually copy them to every machine and teammate. One slip and your credentials are public.

1p-flawless connects your project to [1Password](https://1password.com) so your secrets live in your password manager, not in files on disk. Claude handles the entire setup for you through a guided conversation. You answer a few questions. Everything else is automatic.

---

## How It Works

Think of it like this: instead of your app reading a password from a file, it reads a *pointer*, a safe address that says "go ask 1Password for this value." The actual secret never touches your code or your git history.

```
Before:  DATABASE_URL=postgres://user:mypassword123@host/db   ← in a file, risky
After:   DATABASE_URL={{ op://App Dev/my-app/DATABASE_URL }}  ← a pointer, safe
```

When you run your app, 1Password fills in the real values automatically. Nobody who looks at your code ever sees the actual secrets.

---

## Setup: Claude Does the Work

Just tell Claude: **"set up 1Password for this project"**

Claude launches a silent scan of your entire codebase. Every environment variable your app uses gets discovered automatically. No manual inventory. No guessing. Then it walks you through three quick confirmation steps:

**Step 1: Confirm your vault and project name**
Claude suggests a safe, consistent name for your project's 1Password entry (lowercase, no special characters that would break secret references). You confirm or adjust.

**Step 2: Review your secrets list**
Claude shows every secret it found, sorted by type (database, auth, API keys, etc.). Remove anything you don't need. Add anything it missed.

**Step 3: Choose your injection style**
- **Write to disk** (`op inject`): generates a `.env.local` file on demand. Best for most projects.
- **In-memory only** (`op run`): secrets exist only while your dev server runs, never touch disk. Best for OAuth credentials.
- **Both**: recommended.

After confirmation, Claude creates everything automatically:
- Your 1Password item with all fields
- Template files with safe pointer references
- An `npm run env:inject` script so one command refreshes your `.env.local`
- `.gitignore` entries so secrets can never accidentally commit
- A `npm run dev:auth` script for in-memory injection

---

## Naming Conventions (Automatic)

1Password has a quirk: special characters like `#`, `/`, and spaces in item names silently break secret references. Most people only discover this after hours of debugging.

1p-flawless enforces safe naming automatically:
- Item names derived from your `package.json` name (lowercase, hyphens only)
- One item per project, all secrets as fields on that item
- Field names match your environment variable names exactly

You never have to think about this. The naming conventions are baked in.

---

## Zero Friction Daily Use

After setup, your workflow doesn't change:

```bash
npm run env:inject   # pull latest secrets from 1Password → .env.local
npm run dev:auth     # start dev server with secrets injected in memory
```

Rotate a credential? Tell Claude: **"rotate my Stripe key"** and it updates 1Password and verifies the new value resolves correctly.

Check everything's wired up? Run **`/1p-status`** for a full health check: vault connection, all fields, template status, and npm scripts in one output.

---

## No More Touch ID Prompts

By default, every `op` command triggers a Touch ID prompt. Fine for occasional use, painful during active development.

The `sac-setup` skill creates a service account token in your shell config. After that, `op` commands run silently in the background without interrupting you. One setup, permanently quieter workflow.

---

## CI/CD in Two Minutes

Tell Claude: **"add 1Password secrets to GitHub Actions"**

Claude reads your project's 1Password item, generates the exact workflow step you need, and tells you the one GitHub secret to add. Your CI pipeline pulls secrets from 1Password on every run. No environment variable management in GitHub's settings UI, no secrets drift between local and CI.

---

## Skills & Commands

| | Name | What to say / type |
|---|------|-------------------|
| **Skill** | `setup` | "set up 1Password for this project" |
| **Skill** | `sac-setup` | "stop Touch ID prompts for op CLI" |
| **Skill** | `cicd-setup` | "add my secrets to GitHub Actions" |
| **Command** | `/1p-status` | Health check: vault, fields, templates, scripts |
| **Command** | `/1p-inject` | Generate `.env.local` from your template |
| **Command** | `/1p-rotate [FIELD]` | Update a credential and verify it resolves |

---

## Prerequisites

- [1Password](https://1password.com) account (any plan)
- [1Password CLI (`op`)](https://1password.com/downloads/) installed
- Authenticated: `op signin`

---

## Relationship to 1p-local-auth

`1p-flawless` supersedes `1p-local-auth`. The older plugin covers OAuth dev credentials only. This plugin covers everything: all secret types, both injection styles, naming conventions, SAC setup, and CI/CD. You can keep both installed and they won't conflict.

---

<p align="center">
  <a href="https://vaporforge.dev">
    <img src="https://vaporforge.dev/icon.svg" alt="VaporForge" height="28">
  </a>
  <br>
  Built with <a href="https://vaporforge.dev"><strong>VaporForge</strong></a>
</p>
