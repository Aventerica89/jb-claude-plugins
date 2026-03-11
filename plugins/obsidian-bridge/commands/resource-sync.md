---
description: Sync changelog entries for all 6 platform resource notes in ~/Obsidian-Claude/resources/. Fetches each platform's blog/RSS URL and appends new entries as unchecked tasks.
allowed-tools: [Read, Edit, WebFetch]
---

# Resource Sync

Update the Changelog section of each platform note in `~/Obsidian-Claude/resources/` with new posts from the last 7 days.

## Step 1: Read vault path

Read `~/.claude/obsidian-config.json`. Extract `vault_path`. Default to `~/Obsidian-Claude` if not set.

## Step 2: Process each platform note

For each file in `{vault_path}/resources/`: `1password.md`, `anthropic.md`, `cloudflare.md`, `supabase.md`, `turso.md`, `vercel.md`:

1. Read the file and extract the `rss:` field from YAML frontmatter
2. Fetch that URL via WebFetch — it may be an RSS feed or a blog index page
3. Extract post entries from the last 7 days: title, URL, date, and a one-sentence summary
4. Read the existing `## Changelog` section and collect all URLs already present (to deduplicate)
5. For each new entry not already in the Changelog, prepend it as:
   `- [ ] YYYY-MM-DD — [Title](url) — one sentence summary`
   (newest first, before existing entries)
6. Update the `date:` frontmatter field to today's date
7. Use Edit (not Write) to apply changes — surgical edits only

If WebFetch returns an error or no recent posts, skip that note and continue.

## Step 3: Report

Output a summary table:

```
Resource sync complete — 2026-03-10

1password:   +2 new entries
anthropic:   no new posts
cloudflare:  +3 new entries
supabase:    +1 new entry
turso:       no new posts
vercel:      +2 new entries
```
