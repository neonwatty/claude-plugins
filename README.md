# Claude Code Plugins Marketplace

[![Discord](https://img.shields.io/badge/Discord-Join%20Server-7289da?style=flat&logo=discord&logoColor=white)](https://discord.gg/7xsxU4ZG6A)

Custom plugins for Claude Code.

## Available Plugins

| Plugin | Description |
|--------|-------------|
| [looper](./looper) | Autonomous feature implementation with independent verification and design quality enforcement |

## Installation

### Add This Marketplace

```bash
/plugin marketplace add neonwatty/claude-plugins
```

### Install a Plugin

```bash
/plugin install looper@neonwatty
```

## For Teams

Add to your project's `.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "neonwatty": {
      "source": {
        "source": "github",
        "repo": "neonwatty/claude-plugins"
      }
    }
  },
  "enabledPlugins": [
    "looper@neonwatty"
  ]
}
```

Team members automatically get the plugins when they trust the repo.
