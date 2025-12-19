# Project Name

<!-- Copy this file to your project root as CLAUDE.md and customize -->

## Project Overview

{Brief description of what this project does}

## Tech Stack

- **Framework:** Vite + React
- **Language:** TypeScript
- **Package Manager:** npm

## Looper Configuration

These settings are used by the `/implement` command. All commands use `make` targets:

- **Test Command:** `make test`
- **E2E Test Command:** `make test-e2e`
- **E2E Framework:** Playwright
- **Lint Command:** `make lint`
- **Knip Command:** `make knip`
- **Build Command:** `make build`
- **All Checks:** `make check` (runs lint, knip, test, build)
- **App URL:** `http://localhost:5173`
- **Dev Server Command:** `make dev`

## Key Directories

- `src/components/` - UI components
- `src/pages/` - Page routes
- `src/api/` - API endpoints
- `src/lib/` - Shared utilities
- `e2e/` - Playwright E2E tests

## Patterns to Follow

- Use functional components with hooks
- Strict TypeScript (no `any`)
- All exports must be used (enforced by knip)
- E2E tests for all features

## Makefile Commands

| Command | What It Does |
|---------|--------------|
| `make dev` | Start Vite dev server |
| `make build` | Build for production |
| `make test` | Run Vitest unit tests |
| `make test-e2e` | Run Playwright E2E tests |
| `make lint` | Run ESLint |
| `make lint-fix` | Run ESLint with auto-fix |
| `make knip` | Find dead code/unused exports |
| `make typecheck` | Run TypeScript compiler check |
| `make check` | Run all checks (lint, knip, typecheck, test, build) |
| `make ci` | CI pipeline (same as check + test-e2e) |

## Required Makefile

Create a `Makefile` in your project root:

```makefile
.PHONY: dev build test test-e2e lint lint-fix knip typecheck check ci clean

# Development
dev:
	npm run dev

# Build
build:
	npm run build

# Testing
test:
	npm run test

test-e2e:
	npx playwright test

test-e2e-ui:
	npx playwright test --ui

# Code Quality
lint:
	npx eslint . --ext .ts,.tsx

lint-fix:
	npx eslint . --ext .ts,.tsx --fix

knip:
	npx knip

typecheck:
	npx tsc --noEmit

# Combined Checks
check: lint knip typecheck test build

# CI Pipeline
ci: check test-e2e

# Cleanup
clean:
	rm -rf dist node_modules/.cache
```

## Required Dev Dependencies

```bash
npm install -D eslint @typescript-eslint/eslint-plugin @typescript-eslint/parser
npm install -D knip
npm install -D @playwright/test
npm install -D vitest
```
