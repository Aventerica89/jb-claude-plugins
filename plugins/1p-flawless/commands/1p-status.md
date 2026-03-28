---
description: "1Password health check for the current project — op CLI version, vault auth, item fields (masked), template status, npm scripts."
allowed-tools: Read, Bash
---

# /1p-status

Run a health check for 1p-flawless in the current project.

## Steps

1. Check op CLI:
```bash
op --version 2>/dev/null || echo "NOT INSTALLED"
op vault list 2>/dev/null | head -3
```

2. Detect project slug: read `package.json` `name` field, sanitize to lowercase-hyphenated (replace non-`[a-z0-9]` with `-`, collapse consecutive `-`).

3. Find which vault the project item lives in:
```bash
op item list --vault "App Dev" 2>/dev/null | grep -i "<slug>"
```
If not found in App Dev, check other vaults from `op vault list`.

4. List item fields (masked):
```bash
op item get "<slug>" --vault "<vault>" --format json 2>/dev/null | jq -r '.fields[] | "\(.label)\t= ***"'
```

5. Check templates and scripts:
```bash
ls .env.local.tpl .env.local.dev.tpl 2>/dev/null
grep -E '"env:inject"|"dev:auth"' package.json 2>/dev/null
```

6. Output:

```
1p-flawless status

op CLI:        v<version> — authenticated
Vault:         <vault>
Project item:  <slug> (<N> fields)

Fields (masked):
  FIELD_NAME     = ***
  ...

Templates:
  .env.local.tpl       <exists / missing>  (<N> refs)
  .env.local.dev.tpl   <exists / missing>  (<N> refs)

Scripts:
  env:inject    <found / missing>
  dev:auth      <found / missing>
```

If op is not authenticated: "Run `op signin` first."
If item not found: "No 1P item found for '<slug>'. Run the setup skill to create one."

$ARGUMENTS
