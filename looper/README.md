# Looper

Autonomous feature implementation with independent verification.

```
/implement {feature}
        │
        ▼
   ┌─────────┐
   │ Planner │ → Creates PLAN.md with 2-4 steps
   └────┬────┘
        │
   ┌────▼────┐      ┌─────────┐
   │ Worker  │ ───► │ Checker │  (independent sub-agent)
   │ (main)  │ ◄─── │         │
   └────┬────┘      └─────────┘
        │                │
        │          Uses Chrome MCP for
        │          visual verification
        │
   (if 3 failures)
        ▼
   Escalate to user
```

## Installation

### Option 1: Add Marketplace to Your Project

In your project's `.claude/settings.json`:

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

### Option 2: Install Globally

```bash
/plugin marketplace add neonwatty/claude-plugins
/plugin install looper@neonwatty
```

## Commands

| Command | Purpose |
|---------|---------|
| `/implement {feature}` | Full autonomous implementation with verification loop |
| `/plan {feature}` | Create implementation plan only (no coding) |
| `/check [step]` | Manually verify a step's implementation |
| `/resume [feature]` | Resume in-progress implementation |

## How It Works

1. **Plan** - Breaks feature into 2-4 verifiable steps
2. **Implement** - Builds each step, commits changes
3. **Check** - Independent agent verifies via code review + Chrome MCP visual checks
4. **Loop** - If issues found, fix and re-check (max 3 attempts)
5. **Escalate** - After 3 failures, asks user for guidance
6. **Ship** - Creates PR when all steps pass

## Project Configuration

Copy `CLAUDE.template.md` to your project as `CLAUDE.md` and fill in:

```markdown
## Looper Configuration
- **Test Command:** npm test
- **App URL:** http://localhost:3000
- **Dev Server:** npm run dev
```

## Plans Storage

Plans are stored in `.plans/{feature-name}/PLAN.md` and preserved after completion for reference.

## Requirements

- Chrome MCP server (for visual verification)
- GitHub CLI (`gh`) for PR creation
- Git repository

## Updating

```bash
/plugin update looper@neonwatty
```

## License

MIT
