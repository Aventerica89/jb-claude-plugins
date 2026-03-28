---
description: "1Password health check — op CLI, vault auth, project item fields, templates, scripts."
allowed-tools: Read, Bash
---

Run a complete 1p-flawless health check for this project.

**op CLI:** !`op --version 2>/dev/null || echo "NOT INSTALLED"`
**Vaults:** !`op vault list 2>/dev/null | head -5 || echo "NOT AUTHENTICATED"`
**Project name:** !`cat package.json 2>/dev/null | python3 -c "import sys,json; print(json.load(sys.stdin).get('name','MISSING'))" 2>/dev/null || echo "MISSING"`

If op is NOT INSTALLED or NOT AUTHENTICATED, output "Run \`op signin\` first." and stop.

Compute the project slug from the name above: lowercase, replace non-`[a-z0-9]` with `-`, collapse consecutive `-`, strip leading/trailing `-`.

Run each check using the Bash tool:

1. Find vault item: `op item list --vault "App Dev" 2>/dev/null | grep -i <slug>` — if empty, repeat for each other vault from the list above.
2. List fields masked: `op item get <slug> --vault <vault> --format json 2>/dev/null | jq -r '.fields[] | "\(.label)\t= ***"'`
3. Check templates: `ls -1 .env.local.tpl .env.local.dev.tpl 2>/dev/null`
4. Count template refs: `grep -c 'op://' .env.local.tpl 2>/dev/null || echo 0` and `grep -c 'op://' .env.local.dev.tpl 2>/dev/null || echo 0`
5. Check scripts: `grep -E '"env:inject"|"dev:auth"' package.json 2>/dev/null`

Output this summary:
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

If item not found: "No 1P item found for '<slug>'. Run the setup skill to create one."

$ARGUMENTS
