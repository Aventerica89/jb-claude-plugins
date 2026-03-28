---
description: "Run op inject to generate .env.local from .env.local.tpl. Confirms vars written and warns on any gitignore issues."
allowed-tools: Read, Bash(op:*), Bash(wc:*), Bash(grep:*)
---

# /1p-inject

Generate `.env.local` from `.env.local.tpl` using `op inject`.

## Steps

1. Check `.env.local.tpl` exists:
```bash
ls .env.local.tpl 2>/dev/null || echo "NOT FOUND"
```
If missing: "No `.env.local.tpl` found. Run the `setup` skill first."

2. Run inject:
```bash
op inject -i .env.local.tpl -o .env.local 2>&1
```

3. Verify output:
```bash
wc -l .env.local 2>/dev/null
grep -c '=' .env.local 2>/dev/null
```

4. Check .gitignore — warn if `.env.local` is NOT ignored:
```bash
grep -qE '^\.env\.local$|^\.env\*' .gitignore 2>/dev/null && echo "GITIGNORE OK" || echo "WARNING: .env.local may not be gitignored"
```

5. Output:
```
Injected: .env.local (<N> vars written)

NEXT_PUBLIC_APP_URL = visible (not a secret)
SECRET_VAR          = *** (masked)

Gitignore: OK
```

Common errors:
- `[ERROR] 401` → run `op signin`
- `invalid secret reference` → check `.env.local.tpl` for malformed `op://` paths
- Var is empty but no error → wrong template syntax — `{{ }}` required for op inject (bare `op://` is for `op run` only)

$ARGUMENTS
