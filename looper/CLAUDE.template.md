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
| `make test-e2e` | Run Playwright E2E (Desktop Chrome) |
| `make test-e2e-mobile` | Run E2E on Mobile Chrome + Mobile Safari |
| `make test-e2e-all` | Run E2E on all browsers |
| `make lint` | Run ESLint |
| `make knip` | Find dead code/unused exports |
| `make typecheck` | TypeScript check |
| `make check` | All checks (lint, knip, typecheck, test, build) |
| `make validate` | All checks + E2E tests (run before pushing) |
| `make ci` | Full CI (check + all browser E2E) |

## CI Features

- **3 parallel shards** for desktop E2E tests (3x faster)
- **Separate mobile job** for Mobile Chrome + Mobile Safari
- All checks must pass before E2E runs

**Note:** Looper will create the Makefile, Playwright config, and CI workflow automatically if missing.
