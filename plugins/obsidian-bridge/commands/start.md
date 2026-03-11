---
description: Start all Obsidian background cron jobs for this session — resource-sync every 6h and vault-sync every 12h.
allowed-tools: [CronCreate]
---

# Start Obsidian Background Jobs

Schedule both recurring Obsidian maintenance jobs for this session.

## Jobs to schedule

**Resource Sync** — every 6 hours:
- cron: `0 */6 * * *`
- prompt: `/obsidian-bridge:resource-sync`
- recurring: true

**Vault Sync** — every 12 hours:
- cron: `0 */12 * * *`
- prompt: `/obsidian-bridge:vault-sync`
- recurring: true

## After scheduling

Report the job IDs and confirm:

```
Obsidian background jobs started

resource-sync   every 6h    job XXXXXXXX
vault-sync      every 12h   job YYYYYYYY

Both auto-expire in 3 days. Run /obsidian-bridge:status to check.
Use CronDelete("ID") to cancel early.
```
