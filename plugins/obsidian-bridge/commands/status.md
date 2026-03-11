---
description: Show status of all active Obsidian background cron jobs in this session.
allowed-tools: [CronList]
---

# Obsidian Background Job Status

Call CronList to get all active cron jobs in this session.

Filter for jobs whose prompt contains `obsidian-bridge` and display them in a table:

```
Obsidian background jobs

job        prompt                            cadence     next fire
--------   --------------------------------  ----------  ---------
XXXXXXXX   /obsidian-bridge:resource-sync   every 6h    ~2h
YYYYYYYY   /obsidian-bridge:vault-sync      every 12h   ~4h
```

If no obsidian-bridge jobs are found, output:

```
No Obsidian background jobs running.
Run /obsidian-bridge:start to schedule them.
```
