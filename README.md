# Claude Code Plugins Marketplace

Custom plugins for Claude Code.

## Available Plugins

| Plugin | Description |
|--------|-------------|
| [looper](./looper) | Autonomous feature implementation with independent verification |

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
