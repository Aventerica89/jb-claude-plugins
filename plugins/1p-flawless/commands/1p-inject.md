---
description: "Generate .env.local from .env.local.tpl using op inject. Confirms vars written, warns on gitignore issues."
allowed-tools: Bash
---

Generate `.env.local` from `.env.local.tpl` using `op inject`.

**Template check:** !`ls .env.local.tpl 2>/dev/null && echo "FOUND" || echo "NOT FOUND"`

If template is NOT FOUND, output: "No `.env.local.tpl` found. Run the `setup` skill first." and stop.

Run each step using the Bash tool:

1. Inject: `op inject -i .env.local.tpl -o .env.local 2>&1`
2. Count vars written: `grep -c '=' .env.local 2>/dev/null || echo 0`
3. Read the var names and values: `cat .env.local 2>/dev/null`
4. Gitignore check: `grep -qE '^\.env\.local$|^\.env\*' .gitignore 2>/dev/null && echo "GITIGNORE OK" || echo "WARNING: .env.local may not be gitignored"`

Output a summary listing each variable. Mask the value as `***` for any variable whose name contains SECRET, KEY, TOKEN, PASSWORD, PRIVATE, or API. Show others as-is.

```
Injected: .env.local (<N> vars written)

NEXT_PUBLIC_APP_URL = https://example.com
SECRET_API_KEY      = ***

Gitignore: OK
```

Common errors:
- `[ERROR] 401` — run `op signin`
- `invalid secret reference` — malformed `op://` path in `.env.local.tpl`
- Empty var, no error — wrong template syntax: `{{ }}` required for `op inject`; bare `op://` is for `op run` only

$ARGUMENTS
