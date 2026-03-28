# Chains

Skill orchestration for Claude Code. Bundle multiple skills into structured audit-fix-verify flows.

## What is a Chain?

A **chain** orchestrates multiple existing skills with extra embedded checks into a single structured workflow. Instead of typing the same multi-skill invocations with the same extra instructions every time, invoke one command.

## Available Chains

### `/chains-ui-polish`

Comprehensive UI/UX polish pass on a component cluster. Runs four skill audits plus eight extra checks in a structured flow.

**Delegated Skills** (required, install separately):

| Skill | Author | License | Source |
|-------|--------|---------|--------|
| [baseline-ui](https://www.ui-skills.com/) | [ibelick](https://github.com/ibelick) | See source | UI baseline enforcement |
| [frontend-design](https://www.ui-skills.com/) | [ibelick](https://github.com/ibelick) | See LICENSE.txt | Production-grade frontend design |
| [12-principles-of-animation](https://www.ui-skills.com/) | [raphael-salaja](https://github.com/raphael-salaja) | MIT | Disney's 12 principles for web |
| [fixing-motion-performance](https://www.ui-skills.com/) | [ibelick](https://github.com/ibelick) | See source | Animation performance fixes |

Install from [ui-skills.com](https://www.ui-skills.com/) or [ibelick/ui-skills](https://github.com/ibelick/ui-skills).

**8 Extra Checks** (built into this chain):

1. Toast notification audit
2. Description sufficiency
3. Tooltip needs (shadcn Radix Tooltip)
4. Loading / empty / error states
5. Keyboard navigation + focus order
6. Hover / active / disabled states
7. Responsive behavior (touch targets, safe areas)
8. Design system core (globals.css token scan)

**Phase Flow:**

```
Phase 0  SYSTEM CHECK     Scan design tokens for contrast, font size, spacing issues
Phase 1  INVENTORY        Number every element in the target cluster
Phase 2  AUDIT            Run all 4 skills + 8 extra checks
Phase 3  PRESENT (STOP)   Grouped findings — [UI] and [Motion] — wait for approval
Phase 4a FIX UI           Apply UI fixes one component at a time
Phase 4b FIX MOTION       Apply motion fixes one component at a time
Phase 5  VERIFY           Re-audit for regressions and missed items
Phase 6  RECAP            Per-component changelog with category tags
```

## Install

```bash
claude plugin install chains@jb-claude-plugins
```

**Prerequisites:** The four UI skills above must be installed separately. This plugin orchestrates them — it does not bundle or redistribute them.

## Usage

```
/chains-ui-polish these 3 buttons and the sidebar text
/chains-ui-polish the MCP server card and its action row
/chains-ui-polish the settings form inputs and submit area
```

## Credits

This plugin builds on the excellent work of:

- **[ibelick/ui-skills](https://github.com/ibelick/ui-skills)** — The foundational UI and animation skills that this chain orchestrates. Visit [ui-skills.com](https://www.ui-skills.com/) to install them.
- **[raphael-salaja](https://github.com/raphael-salaja)** — Author of the 12 Principles of Animation skill.

Chains does not copy, modify, or redistribute these skills. It provides an orchestration layer that invokes them by name.

## Creating New Chains

See [chains/conventions.md](https://github.com/Aventerica89/jb-ops/blob/main/chains/conventions.md) for the framework pattern.

---

<p align="center">
  <a href="https://vaporforge.dev">
    <img src="https://vaporforge.dev/icon.svg" alt="VaporForge" height="28">
  </a>
  <br>
  Built with <a href="https://vaporforge.dev"><strong>VaporForge</strong></a>
</p>
