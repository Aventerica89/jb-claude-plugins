---
description: "Update a credential field in the project's 1Password item and verify it resolves."
argument-hint: [FIELD_NAME]
allowed-tools: Bash, AskUserQuestion
---

Rotate (update) a credential field in this project's 1Password item.

**Project name:** !`cat package.json 2>/dev/null | python3 -c "import sys,json; print(json.load(sys.stdin).get('name','MISSING'))" 2>/dev/null || echo "MISSING"`

Compute the slug from the name above: lowercase, replace non-`[a-z0-9]` with `-`, collapse consecutive `-`, strip leading/trailing `-`.

**Field to rotate:** $ARGUMENTS

If no field name was provided, run `op item get <slug> --vault "App Dev" --format json 2>/dev/null | jq -r '.fields[].label'` to list available fields, then ask: "Which field do you want to rotate?"

Use AskUserQuestion to prompt: "Enter the new value for `<FIELD_NAME>`:"

Once the user provides the new value, run each step using the Bash tool:

1. Update the field: `op item edit <slug> --vault "App Dev" "<FIELD_NAME>[password]=<new_value>" 2>&1`
   (See `@${CLAUDE_PLUGIN_ROOT}/skills/setup/references/vault-ops.md` for field type syntax)
2. Verify it resolves: `op read "op://App Dev/<slug>/<FIELD_NAME>" 2>/dev/null | wc -c`

If `wc -c` returns a non-zero number, output:
```
Rotated: <FIELD_NAME> in <slug> (App Dev)
Verified: resolves successfully
```

If `wc -c` returns 0 or the read errors: "Rotation may have succeeded but `op read` failed. Check `op signin` status and try `/1p-status`."
