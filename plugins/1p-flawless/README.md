# 1p-flawless

Complete 1Password workflow plugin for Claude Code. Covers vault setup, secret injection, service accounts, and CI/CD.

## Skills

| Skill | Trigger phrases |
|-------|----------------|
| `setup` | "set up 1Password for this project", "wire 1p secrets", "create my .env template" |
| `sac-setup` | "stop Touch ID prompts for op", "add OP_SERVICE_ACCOUNT_TOKEN" |
| `cicd-setup` | "add 1P secrets to GitHub Actions", "wire load-secrets-action" |

## Commands

| Command | Description |
|---------|-------------|
| `/1p-status` | Health check — vault auth, item fields, templates, scripts |
| `/1p-inject` | Generate `.env.local` from `.env.local.tpl` |
| `/1p-rotate` | Update a credential field in 1P |

## Quick Start

1. Run the `setup` skill in any project
2. It scans for env vars, creates a 1P item, writes templates, patches `package.json` and `.gitignore`
3. Use `npm run env:inject` to generate `.env.local` or `npm run dev:auth` for in-memory OAuth injection

## Prerequisites

- `op` CLI installed: `https://1password.com/downloads/mac/`
- Authenticated: `op signin`

## Agents

| Agent | Purpose |
|-------|---------|
| `secrets-scanner` | Autonomous project scan — discovers vars, checks vaults, returns structured report |

## Relationship to 1p-local-auth

`1p-flawless` supersedes `1p-local-auth`. The older plugin covers OAuth dev credentials only (op run pattern). This plugin covers the full workflow: both op inject and op run, all secret types, SAC setup, and CI/CD. You can keep `1p-local-auth` installed alongside — it won't conflict.

---

<p align="center">
  <a href="https://vaporforge.dev">
    <img src="https://vaporforge.dev/icon.svg" alt="VaporForge" height="28">
  </a>
  <br>
  Built with <a href="https://vaporforge.dev"><strong>VaporForge</strong></a>
</p>
