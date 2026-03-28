# Extra Checks — Detailed Audit Criteria

Reference for the 8 embedded checks in `/chains-ui-polish`. Each check includes what to look for, pass/fail examples, and output format.

## 1. Toast Audit

**Trigger actions**: save, delete, copy, submit, toggle, connect, disconnect, create, update, remove, send, publish, archive, restore, import, export.

**Pass**: Action triggers visible feedback (toast, inline confirmation, or status change).
**Fail**: Action completes silently — no visual confirmation of success or failure.

**Output format**:
```
[N] Component — [Toast] "save" action has no success feedback
  Suggested: toast({ title: "Settings saved" })
  Error case: toast({ title: "Failed to save", variant: "destructive" })
```

**Do not flag**: Navigation actions, open/close toggles with visible state change, focus shifts.

## 2. Description Sufficiency

**Check**: Every label, heading, placeholder, helper text, and aria-label.

**Pass**: Text is specific, uses plain language, and describes what the element does or contains.
**Fail**: Text is vague ("Settings"), uses internal jargon ("MCP config"), or is missing entirely.

**Examples**:
- Fail: placeholder="Enter value" — what value?
- Pass: placeholder="e.g., https://api.example.com"
- Fail: label="Config" — config of what?
- Pass: label="Server URL"

**Output format**:
```
[N] Component — [Desc] label "Config" is vague
  Rewrite: "MCP Server Configuration"
```

## 3. Tooltip Needs

**Elements needing tooltips**: icon-only buttons (no visible label), abbreviated text, toggles without inline explanation, status indicators (colored dots, badges without legend).

**Standard component**: shadcn Radix Tooltip (`npx shadcn@latest add tooltip`).

**Pass**: Element has a visible label OR a tooltip with descriptive text.
**Fail**: Icon-only button with no tooltip and no aria-label, or toggle with no explanation of what it controls.

**Output format**:
```
[N] Component — [Tooltip] icon-only delete button, no context
  Suggested: <Tooltip><TooltipTrigger>...</TooltipTrigger><TooltipContent>Delete server</TooltipContent></Tooltip>
```

## 4. Loading / Empty / Error States

**Check each component that**:
- Fetches data (API call, database query, subscription)
- Triggers async operations (save, delete, connect)
- Displays a collection that could be empty

**Three states required**:
1. **Loading**: Spinner, skeleton, or shimmer while waiting
2. **Empty**: Meaningful message + optional action (not just blank space)
3. **Error**: Inline error message with retry option (not silent failure)

**Output format**:
```
[N] Component — [State] missing loading state for async fetch
[N] Component — [State] empty collection shows blank space, no empty state
[N] Component — [State] API error silently swallowed, no user feedback
```

## 5. Keyboard Navigation + Focus

**Tab order check**: Follow the DOM order. Does tab move through the cluster in a logical reading order (left-to-right, top-to-bottom)?

**Focus ring check**: Every focusable element must show a visible focus indicator. Check for `outline: none` or `outline: 0` without a replacement.

**Escape behavior**: Any overlay, dropdown, or modal within the cluster must close on Escape and return focus to the trigger.

**Arrow key groups**: Radio groups, tab lists, and menus should use arrow keys for internal navigation (not Tab).

**Output format**:
```
[N] Component — [A11y] focus ring suppressed (outline: none, no replacement)
[N] Component — [A11y] tab order: [3] -> [1] -> [2], expected [1] -> [2] -> [3]
[N] Component — [A11y] dropdown does not close on Escape
```

## 6. Hover / Active / Disabled States

**Four states per interactive element**:

| State | Visual cue |
|-------|-----------|
| Default | Base appearance |
| Hover | Color shift, underline, background change, or shadow |
| Active (pressed) | Scale transform (0.95-0.97) or color/shadow inversion |
| Disabled | `opacity-50` + `cursor-not-allowed` + `pointer-events-none` |

**Pass**: All four states are visually distinct.
**Fail**: Missing any state, or hover and active look identical.

**Output format**:
```
[N] Component — [States] missing :active state (no scale or color shift on press)
[N] Component — [States] missing :disabled style (no visual distinction)
```

## 7. Responsive Behavior

**Touch targets**: Every interactive element must have a minimum tap area of 44x44px. Use `min-h-[44px] min-w-[44px]` or padding/pseudo-element expansion for visually smaller elements.

**Stacking**: At mobile breakpoints (< 640px), check that horizontal layouts stack vertically where appropriate. Buttons should not overflow off-screen.

**Safe areas**: Edge-anchored elements (fixed bottom bar, sticky header, floating action button) must include `env(safe-area-inset-*)` padding.

**Output format**:
```
[N] Component — [Responsive] tap target 32x28px, below 44x44px minimum
[N] Component — [Responsive] button row overflows at 375px viewport
[N] Component — [Responsive] fixed bottom bar missing safe-area-inset-bottom
```

## 8. Design System Core

Runs in Phase 0, not Phase 2. Included here for completeness.

**Font size floor**: Any CSS custom property, Tailwind config value, or base style producing text below 12px. Exception: legal fine print, timestamps in data-dense tables.

**Contrast ratio**: Compute resolved hex values for every text-color / background-color pair in the token system. Flag pairs below:
- 4.5:1 for normal text (< 18px or < 14px bold)
- 3:1 for large text (>= 18px or >= 14px bold)

**Spacing extremes**: Padding or gap tokens below 4px on any element that receives user interaction (buttons, inputs, links, toggles).

**Output format**:
```
[SYSTEM] --font-size-xs: 9px → below 12px floor
[SYSTEM] --color-muted-foreground on --background: 2.8:1 → below 4.5:1 AA
[SYSTEM] --spacing-xs: 2px → below 4px floor on interactive elements
```
