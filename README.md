# JB Claude Plugins

A collection of Claude Code plugins for professional developer workflows — secrets management, knowledge browsing, and context efficiency.

## Plugins

| Plugin | Description |
|--------|-------------|
| [1p-flawless](plugins/1p-flawless) | Complete 1Password workflow — vault setup, `op inject`, `op run`, SAC, CI/CD |
| [1p-local-auth](plugins/1p-local-auth) | Local dev OAuth credentials via 1Password — test any auth provider on any branch |
| [obsidian-bridge](plugins/obsidian-bridge) | Zero-copy Obsidian vault over `~/.claude/` — browse Claude's full knowledge base in Obsidian |
| [skillforge](plugins/skillforge) | Deferred skill loading — compress skills to stubs, save ~88% per-turn token overhead |

## Install the Marketplace

```bash
claude plugin marketplace add Aventerica89/jb-claude-plugins
```

## Install a Plugin

```bash
claude plugin install 1p-flawless@jb-claude-plugins
claude plugin install 1p-local-auth@jb-claude-plugins
claude plugin install obsidian-bridge@jb-claude-plugins
claude plugin install skillforge@jb-claude-plugins
```

If you see SSH errors during install:

```bash
git config --global url."https://github.com/".insteadOf git@github.com:
```

## Requirements

- [Claude Code](https://claude.ai/code) CLI
- Plugin-specific requirements listed in each plugin's README

---

<p align="center">
  <a href="https://vaporforge.dev">
    <img src="https://vaporforge.dev/icon.svg" alt="VaporForge" height="28">
  </a>
  <br>
  Built with <a href="https://vaporforge.dev"><strong>VaporForge</strong></a>
</p>
