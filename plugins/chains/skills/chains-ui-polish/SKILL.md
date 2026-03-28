---
name: chains-ui-polish
description: This skill should be used when the user asks to "polish UI components", "run a UI/UX audit on this cluster", "chains-ui-polish", "audit and fix these buttons/inputs/cards", or wants a comprehensive UI + motion + accessibility pass on a group of related elements. Orchestrates /baseline-ui, /frontend-design, /12-principles-of-animation, and /fixing-motion-performance with 8 extra checks in a structured audit-present-fix-verify-recap flow.
---

# Chains: UI Polish

Orchestrate a comprehensive UI/UX polish pass on a component cluster. Chain four specialized skills with eight extra audit checks into a single structured flow: inventory, audit, present, fix, verify, recap.

## Scope

Target: a user-specified **component cluster** (a group of related elements, not a full page). Examples: "these 3 buttons and the sidebar text", "the MCP server card and its action row", "the settings form inputs and submit area".

## Phase Flow

Execute phases in strict order. Do not skip phases. Do not combine phases.

### Phase 0 — SYSTEM CHECK

Scan the project's design token source (typically `globals.css`, `tailwind.config`, or equivalent) for system-level issues that would cascade to every component:

- **Font size floor**: Flag any token or base style producing text smaller than 12px
- **Contrast ratio**: Flag text/background color pairs below WCAG AA (4.5:1 for normal text, 3:1 for large text). Compute from the resolved hex values.
- **Spacing extremes**: Flag tokens producing padding or gap below 4px on interactive elements

If issues are found, print them and ask:

> "These design system issues affect all components. Fix at the system level before auditing the cluster?"

Apply approved system fixes. If no issues, state "System check passed" and proceed.

### Phase 1 — INVENTORY

Read the target files identified by the user. List every distinct interactive and visual element in the cluster. Number each one:

```
Component Inventory:
[1] Save button
[2] Cancel button
[3] Description text
[4] Status badge
[5] Dropdown menu trigger
```

Print the inventory. This numbering is used for all subsequent findings and the final recap.

### Phase 2 — AUDIT

Run all audits sequentially, collecting findings tagged with component numbers and categories.

#### Skill Audits

Invoke each skill against the target files, applying its full ruleset:

a. **`/baseline-ui`** — stack compliance, accessible primitives, interaction patterns, layout, performance
b. **`/frontend-design`** — typography, color cohesion, spatial composition, visual quality
c. **`/12-principles-of-animation`** — timing, easing, physics, staging rules
d. **`/fixing-motion-performance`** — compositor props, `prefers-reduced-motion`, layout thrashing, scroll-linked motion

#### Extra Checks

After skill audits, run these eight embedded checks against the same cluster:

**1. Toast Audit**
Identify user-initiated actions (save, delete, copy, submit, toggle, connect, disconnect) that lack success or error feedback. For each, suggest a toast message. Do not flag actions that already show inline feedback.

**2. Description Sufficiency**
Read every label, heading, placeholder, and helper text in the cluster. Flag text that is vague, uses internal jargon, or would confuse a first-time user. Propose concrete rewrites.

**3. Tooltip Needs**
Identify elements that need contextual help but lack it: icon-only buttons, abbreviated labels, toggles without explanation, status indicators without legend. Standard component: **shadcn Radix Tooltip**. Suggest tooltip text for each.

**4. Loading / Empty / Error States**
For any component that fetches data or triggers an async operation, check: does it handle loading (spinner or skeleton), empty (meaningful empty state), and error (inline message or retry)? Flag missing states.

**5. Keyboard Navigation + Focus**
Trace the tab order through the cluster. Check: logical sequence, visible focus rings on every focusable element, Escape closes overlays, arrow keys navigate within groups (radio, tabs, menu). Report the focus order map and any violations.

**6. Hover / Active / Disabled States**
For every interactive element, check for four distinct visual states: default, hover, active (pressed), and disabled. Flag elements missing any state. Active state must include a scale or color shift per `/baseline-ui` rules.

**7. Responsive Behavior**
Check touch targets (minimum 44x44px tap area), element stacking/reflow at mobile breakpoints, and `env(safe-area-inset-*)` on edge-anchored elements. Flag violations per the mobile-web enforcement rules.

**8. Design System Core**
(Already executed in Phase 0. Include any remaining Phase 0 findings that were deferred.)

Tag every finding with its component number and category:

```
[3] Description text — [UI] contrast ratio 2.8:1, below WCAG AA 4.5:1
[1] Save button — [Motion] timing 450ms, exceeds 300ms limit (timing-under-300ms)
[5] Dropdown trigger — [Tooltip] icon-only button, no accessible label or tooltip
```

### Phase 3 — PRESENT (STOP HERE)

Group all findings into two sections:

**[UI Findings]** — from `/baseline-ui`, `/frontend-design`, and extra checks 1-7
**[Motion Findings]** — from `/12-principles-of-animation`, `/fixing-motion-performance`

Print a per-component summary table:

```
| # | Component         | UI | Motion | Extra | Total |
|---|-------------------|----|--------|-------|-------|
| 1 | Save button       |  2 |      1 |     1 |     4 |
| 2 | Cancel button     |  1 |      0 |     1 |     2 |
| 3 | Description text  |  3 |      0 |     0 |     3 |
```

Then print the full finding details grouped by [UI] and [Motion].

**STOP. Wait for user approval before proceeding to fixes.** Ask:

> "N findings across M components. Approve all fixes, or specify which to skip?"

### Phase 4a — FIX UI

Apply UI fixes: contrast, spacing, typography, interaction states, toasts, tooltips, descriptions, loading/empty/error states, keyboard navigation, responsive adjustments.

Work one component at a time. Do not batch multiple components into a single edit.

### Phase 4b — FIX MOTION

Apply motion fixes: timing corrections, easing updates, spring replacements, compositor property enforcement, `prefers-reduced-motion` guards, stagger limits.

Work one component at a time.

### Phase 5 — VERIFY

Run a quick re-audit: apply all four skill rulesets and all eight extra checks against the modified files. Report:

- **Resolved**: count of original findings now passing
- **Remaining**: any original findings not yet addressed (with reason)
- **New**: any issues introduced by the fixes

Fix any newly discovered issues. If new issues cascade, limit to one additional fix cycle.

### Phase 6 — RECAP

Print the final per-component changelog using this exact format:

```
## /chains-ui-polish Recap

### System Fixes Applied
- globals.css: <what changed>

### Component Changelog

[1] Save Button
  [UI]     + <change description>
  [Motion] + <change description>
  [Toast]  + <change description>

[2] Cancel Button
  [UI]     + <change description>
  [Tooltip]+ <change description>
  [Motion] ~ No motion issues found

### Summary
Found: N issues (X UI, Y Motion, Z Extra)
Fixed: N | Deferred: N (reason)
Verify: N new issues in re-audit (fixed/remaining)
```

Use `+` for additions/fixes, `~` for "no issues found" in that category, `-` for removals.

## Rules

- Never skip a phase. If a phase produces zero findings, state that and proceed.
- Always use the numbered component inventory established in Phase 1 — never renumber mid-flow.
- Phase 3 is a hard stop. Do not auto-fix without user approval.
- Fix UI before Motion (Phase 4a before 4b) — UI changes may resolve some motion findings.
- The verify pass (Phase 5) is mandatory, not optional. Skipping it defeats the purpose.
- Keep the recap (Phase 6) factual. Do not editorialize. List what changed per component.

## Additional Resources

### Reference Files

- **`references/extra-checks.md`** — detailed audit criteria for each of the 8 extra checks with pass/fail examples

### Delegated Skills

This skill delegates to four existing skills. Do not duplicate their rules — invoke them:
- `/baseline-ui` — stack, components, interaction, layout, performance
- `/frontend-design` — typography, color, spatial, visual quality
- `/12-principles-of-animation` — timing, easing, physics, staging
- `/fixing-motion-performance` — compositor, reduced-motion, layout thrashing
