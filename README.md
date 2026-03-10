# JB Claude Plugins

Claude Code plugin marketplace for JBMD Creations.

## Install

```bash
claude plugin marketplace add Aventerica89/jb-claude-plugins
```

## Available Plugins

| Plugin | Description |
|--------|-------------|
| [1p-local-auth](plugins/1p-local-auth) | Manage local dev OAuth credentials via 1Password |
| [obsidian-bridge](plugins/obsidian-bridge) | Zero-copy Obsidian vault over ~/.claude/ via symlinks |

## Install a Plugin

```bash
# Install one
claude plugin install 1p-local-auth@jb-claude-plugins
claude plugin install obsidian-bridge@jb-claude-plugins

# If SSH errors occur during install
git config --global url."https://github.com/".insteadOf git@github.com:
```
