---
description: "Update a specific credential in the project's 1Password item and verify injection still works."
argument-hint: [FIELD_NAME]
---

# /1p-rotate

Update a credential field in the project's 1P item.

## Steps

1. Get project slug from `package.json` `name` (sanitize to lowercase-hyphenated).

2. If no field name provided in $ARGUMENTS: list current fields and ask which to rotate:
```bash
op item get "<slug>" --vault "App Dev" --format json 2>/dev/null | jq -r '.fields[].label'
```

3. Prompt user: "Enter the new value for `<FIELD_NAME>`:"

4. Update the field:
```bash
op item edit "<slug>" --vault "App Dev" "<FIELD_NAME>[password]=<new_value>" 2>&1
```
See `references/vault-ops.md` for exact `op item edit` field syntax.

5. Verify the new value resolves:
```bash
op read "op://App Dev/<slug>/<FIELD_NAME>" 2>/dev/null | wc -c
```
Non-zero char count = value exists and is readable.

6. Output:
```
Rotated: FIELD_NAME in <slug> (App Dev)
Verified: resolves successfully
```

If verify fails: "Rotation may have succeeded but `op read` failed. Check `op signin` status and try `/1p-status`."

$ARGUMENTS
