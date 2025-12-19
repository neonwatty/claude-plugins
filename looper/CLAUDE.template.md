# Project Name

## Looper Configuration

Only one setting needed - the dev server URL:

- **App URL:** `http://localhost:5173`

Everything else is standardized via Makefile (created automatically if missing).

## Standard Commands (via Makefile)

| Command | What It Does |
|---------|--------------|
| `make dev` | Start dev server |
| `make build` | Build for production |
| `make test` | Run unit tests |
| `make test-e2e` | Run Playwright E2E tests |
| `make lint` | Run ESLint |
| `make knip` | Find dead code/unused exports |
| `make typecheck` | TypeScript check |
| `make check` | All checks (lint, knip, typecheck, test, build) |
| `make ci` | Full CI (check + test-e2e) |

**Note:** Looper will create the Makefile and install dependencies automatically if missing.
