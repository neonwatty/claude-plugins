# /implement - Autonomous Feature Implementation

Implement a feature with independent verification at each step.

## Arguments
- $ARGUMENTS: The feature description

---

## Configuration

Before starting, confirm the **App URL** for visual verification:
- **App URL:** `http://localhost:5173` (Vite default) or ask user

All other commands are standardized via Makefile:
- `make ci` - runs all checks + E2E tests
- `make dev` - starts dev server

**Note:** Makefile and dependencies are created automatically in Phase 3 if missing.

---

## Phase 1: Planning

### 1.1 Create Feature Branch and Plan Directory

```bash
# Derive feature slug from description
FEATURE_SLUG=$(echo "$ARGUMENTS" | tr ' ' '-' | tr '[:upper:]' '[:lower:]' | cut -c1-50)

# Create feature branch
git checkout -b feature/${FEATURE_SLUG}

# Create plan directory
mkdir -p .plans/${FEATURE_SLUG}
```

### 1.2 Explore Codebase (PARALLEL AGENTS)

Spawn exploration agents in parallel to understand the codebase faster:

```
# PARALLEL AGENT 1: Project Structure Explorer
Task(
  subagent_type: "Explore",
  description: "Explore project structure",
  prompt: """
Explore this codebase and report on:
1. Project structure (key directories)
2. Tech stack (framework, language, package manager)
3. Existing patterns and conventions
4. Test setup (E2E framework, test locations)
5. CI/CD configuration

Return a structured summary.
"""
)

# PARALLEL AGENT 2: Related Code Explorer
Task(
  subagent_type: "Explore",
  description: "Find related code for feature",
  prompt: """
Find code related to this feature: {$ARGUMENTS}

Search for:
1. Similar existing features
2. Files that will need modification
3. Components/modules to integrate with
4. Existing tests to reference

Return file paths and relevant code patterns.
"""
)

# PARALLEL AGENT 3: Dependency Explorer
Task(
  subagent_type: "Explore",
  description: "Analyze dependencies",
  prompt: """
Analyze dependencies for this feature: {$ARGUMENTS}

Identify:
1. External packages needed
2. Internal modules to import
3. API endpoints or services to use
4. Database models or schemas involved

Return dependency map.
"""
)
```

Wait for all exploration agents to complete, then proceed to planning.

### 1.3 Spawn Planner Agent

Use the Task tool to spawn the Planner with exploration results:

```
Task(
  subagent_type: "general-purpose",
  description: "Plan feature implementation",
  prompt: """
You are a planning agent. Create a step-by-step implementation plan. Do NOT implement anything.

## Feature Request
{$ARGUMENTS}

## Codebase Context (from exploration)
{exploration_agent_1_results}
{exploration_agent_2_results}
{exploration_agent_3_results}

## Plan Location
Create the plan at: .plans/{feature_slug}/PLAN.md

## Your Task

1. **Use the exploration results** - don't re-explore, use what was found

2. **Break down** the feature into 2-4 discrete steps where:
   - Each step is independently verifiable
   - Each step can be committed separately
   - Steps are ordered by dependency
   - Each step includes E2E test requirements

3. **Create PLAN.md** at the specified location with this structure:

```markdown
# Implementation Plan: {Feature Title}

## Metadata
- **Status:** IN_PROGRESS
- **Current Step:** 1
- **Created:** {YYYY-MM-DDTHH:MM:SS}
- **Completed:** -
- **Branch:** feature/{slug}
- **PR:** -

## Overview
{2-3 sentence description}

## Steps

### Step 1: {Title}
- **Status:** NOT_STARTED
- **Attempts:** 0
- **Started:** -
- **Completed:** -

**Description:** {What to implement - be specific}

**Files to Change:**
- `path/to/file` - {what change}

**Acceptance Criteria:**
- [ ] {Specific verifiable criterion}
- [ ] {Specific verifiable criterion}

**Visual Check:** {What to verify in browser, or "N/A"}

**Tests:** {Test command or "Run full suite"}

**Checker Feedback:** -

---

### Step 2: {Title}
... (same structure)

---

## Risks
- {Edge cases or considerations}

## Completion Summary
-
```

## Guidelines
- Be specific: "Add submit button to LoginForm that calls handleSubmit" not "Add a button"
- Keep steps small: 15-60 minutes of work each
- Visual checks matter: Checker will use Chrome to verify
- 2-4 steps ideal; more means feature should be split
"""
)
```

### 1.4 Review Plan

After Planner creates PLAN.md, review it before proceeding. Confirm the steps make sense.

---

## Phase 2: Step Execution

For each step in PLAN.md, execute this loop:

```
attempts = 0
max_attempts = 3

while attempts < max_attempts:

    ### 2.1 Implement & Write Tests (PARALLEL AGENTS)

    Spawn TWO agents in parallel using a single message with multiple Task tool calls:

    ```
    # PARALLEL AGENT 1: Implementation
    Task(
      subagent_type: "general-purpose",
      description: "Implement step {n}",
      prompt: """
      Implement this step of the feature.

      ## Step Details
      **Title:** {step_title}
      **Description:** {step_description}
      **Files to Change:** {files from PLAN.md}
      **Acceptance Criteria:** {criteria}

      ## Your Task
      1. Read the existing code in the files to change
      2. Implement the changes described
      3. Follow existing code patterns and conventions
      4. Commit: git commit -m "Step {n}: {title}"

      ## Design Aesthetics (for UI work)
      If this step involves UI, make unexpected, distinctive design choices:
      - **Typography:** Avoid Inter/Roboto/system fonts. Use distinctive typefaces with extreme weight contrasts (100 vs 900) and 3x+ size jumps.
      - **Color:** Avoid purple gradients and corporate palettes. Commit to a cohesive theme via CSS variables. Draw from cultural aesthetics or IDE themes (Dracula, Nord, Tokyo Night).
      - **Motion:** Add staggered reveals, micro-interactions, and smooth transitions. Use animation-delay for cascading effects.
      - **Backgrounds:** Avoid solid white/gray. Use subtle gradients, geometric patterns, or layered depth.

      Reference `looper/guides/design-aesthetics.md` for detailed guidance.

      Return a summary of changes made and files modified.
      """
    )

    # PARALLEL AGENT 2: E2E Test Writing
    Task(
      subagent_type: "general-purpose",
      description: "Write E2E tests for step {n}",
      prompt: """
      Write E2E tests for this feature step. E2E tests are the PRIMARY validation.

      ## Step Details
      **Title:** {step_title}
      **Description:** {step_description}
      **Acceptance Criteria:** {criteria}
      **Visual Check:** {visual_check}
      **E2E Framework:** {from CLAUDE.md config}

      ## Your Task
      1. Understand what the feature does from the step description
      2. Write E2E tests that simulate real user behavior
      3. Cover the main user flow
      4. Cover edge cases and error states
      5. Follow project's E2E test patterns
      6. Commit: git commit -m "Step {n}: Add E2E tests for {title}"

      **Test Location:** e2e/{feature-name}.spec.ts

      Return the test file path and what scenarios are covered.
      """
    )
    ```

    Wait for both agents to complete, then proceed.

    ### 2.2 Run Local Checks
    Run all checks using Makefile:
    ```bash
    make check    # lint, knip, typecheck, test, build
    make test-e2e # E2E tests (primary validation)
    ```

    Or run full CI locally:
    ```bash
    make ci       # runs check + test-e2e
    ```

    - If any check fails: fix issues, attempts++, continue loop

    ### 2.3 Spawn Verification Agents (PARALLEL)

    Spawn verification agents in parallel using a single message with multiple Task tool calls:

    ```
    # PARALLEL AGENT 1: Code Review & Test Verification
    Task(
      subagent_type: "general-purpose",
      description: "Code review step {n}",
      prompt: """
You are a code reviewer. Verify the implementation and test coverage.

## Step Being Verified
**Title:** {step_title}
**Description:** {step_description}
**Acceptance Criteria:** {criteria}

## Changes Made
```diff
{git diff HEAD~2..HEAD}
```

## Test Results
```
{output of test command}
```

## Your Task

### 1. Code Review
- [ ] Logic correctly implements what's described
- [ ] No obvious bugs or broken logic
- [ ] Edge cases handled appropriately

### 2. E2E Test Coverage (PRIORITY)
- [ ] E2E tests exist for the new feature
- [ ] E2E tests cover main user flow
- [ ] E2E tests cover edge cases
- [ ] E2E tests pass

**REJECT if no E2E tests exist.**

## Output
APPROVED or NEEDS_WORK with specific issues.
"""
    )

    # PARALLEL AGENT 2: Visual Verification (if Visual Check is not "N/A")
    Task(
      subagent_type: "general-purpose",
      description: "Visual verification step {n}",
      prompt: """
You are a visual QA checker. Verify the UI implementation using Chrome.

## Step Being Verified
**Title:** {step_title}
**Visual Check:** {visual_check_criteria}
**App URL:** {app_url}

## Your Task
Use Chrome MCP tools to verify:

1. Navigate: mcp__claude-in-chrome__navigate to App URL
2. Screenshot: mcp__claude-in-chrome__computer with action: screenshot
3. Read page: mcp__claude-in-chrome__read_page
4. Find elements: mcp__claude-in-chrome__find for specific elements
5. Test interactions: mcp__claude-in-chrome__computer with action: left_click
6. Verify state: mcp__claude-in-chrome__javascript_tool if needed

## Design Anti-Pattern Detection
Flag if the UI shows signs of "distributional convergence" (generic defaults):
- **Typography:** Inter, Roboto, Open Sans, Lato, or system-ui fonts
- **Color:** Purple/blue gradients, corporate blue (#3b82f6), generic gray palettes
- **Motion:** No animations or transitions on interactive elements
- **Backgrounds:** Solid white (#ffffff) or light gray (#f5f5f5) with no depth

If detected, mark as NEEDS_WORK and request distinctive alternatives per `looper/guides/design-aesthetics.md`.

## Output
APPROVED or NEEDS_WORK with specific visual issues found.
"""
    )
    ```

    Wait for both agents to complete. Both must APPROVE to proceed.

    ---

    **LEGACY SINGLE-AGENT FALLBACK (if Visual Check is "N/A"):**

    If no visual verification needed, spawn single Checker:

    Task(
      subagent_type: "general-purpose",
      description: "Verify step implementation",
      prompt: """
You are an independent QA checker. Verify the implementation is functionally correct. You have fresh context - be objective.

## Step Being Verified
**Title:** {step_title from PLAN.md}
**Description:** {step_description}
**Acceptance Criteria:** {criteria from PLAN.md}
**Visual Check:** {visual_check from PLAN.md}

## Changes Made
```diff
{output of: git diff HEAD~1}
```

## Test Results
```
{output of test command}
```

## App URL
{configured app URL}

---

## Your Task

### 1. Code Review (Functional Focus)
Read the changed files and verify:
- Logic correctly implements what's described
- No obvious bugs or broken logic
- Edge cases handled appropriately

Do NOT nitpick style, formatting, or naming.

### 2. Test Coverage Verification (E2E PRIORITY)
Verify E2E tests were written - this is the primary validation:
- [ ] **E2E tests exist for the new feature** (REQUIRED for all features, not just visual ones)
- [ ] E2E tests cover the main user flow and edge cases
- [ ] E2E tests run successfully and validate real behavior
- [ ] Unit tests exist for complex algorithms or utility functions (secondary)

Use Glob to find test files: `e2e/**/*.spec.ts`, `**/*.test.ts`, `**/__tests__/**`

**REJECT if no E2E tests exist for the feature** - unit tests alone are insufficient.

### 3. Visual Verification (Required when Visual Check is not "N/A")
Use Chrome MCP tools:

1. Navigate: mcp__claude-in-chrome__navigate to App URL
2. Screenshot: mcp__claude-in-chrome__computer with action: screenshot
3. Read page: mcp__claude-in-chrome__read_page
4. Find elements: mcp__claude-in-chrome__find for specific elements
5. Test interactions: mcp__claude-in-chrome__computer with action: left_click
6. Run JS checks: mcp__claude-in-chrome__javascript_tool if needed

### 3.5 Design Anti-Pattern Detection
Flag if the UI shows "distributional convergence" (generic defaults):
- **Typography:** Inter, Roboto, Open Sans, Lato, or system-ui fonts
- **Color:** Purple/blue gradients, corporate blue (#3b82f6), generic gray palettes
- **Motion:** No animations or transitions on interactive elements
- **Backgrounds:** Solid white (#ffffff) or light gray (#f5f5f5) with no depth

If detected, mark as NEEDS_WORK and request distinctive alternatives per `looper/guides/design-aesthetics.md`.

### 4. Decision

**APPROVE if:**
- All acceptance criteria met
- E2E tests exist and pass for the new feature
- E2E tests validate the actual user flow works
- Visual check confirms expected UI (if applicable)
- No functional bugs

**REJECT if:**
- Any criterion not met
- No E2E tests for the feature (unit tests alone are NOT sufficient)
- E2E tests fail
- Visual shows incorrect UI
- Functional bug found
- Design shows distributional convergence (generic fonts, purple gradients, no motion, solid backgrounds)

---

## Output Format

Respond with EXACTLY one of:

### If Approved:
```
APPROVED

## Verification Summary
### Code Review
✓ {what you verified}

### Test Coverage (E2E Priority)
✓ E2E tests verify: {feature user flows}
✓ E2E tests cover edge cases: {what edge cases}
✓ Unit tests (if needed): {complex logic tested}

### Visual Check
✓ {what you saw}
Screenshot confirms: {description}

### Design Quality
✓ Typography: {distinctive font choice, not generic}
✓ Color: {cohesive theme, not purple gradient}
✓ Motion: {animations/transitions present}
✓ Background: {depth/texture, not solid white}

### Functional Check
✓ {what you tested}

All criteria met. Ready to proceed.
```

### If Needs Work:
```
NEEDS_WORK

## Issues Found

### Issue 1: {title}
**Location:** `file.ts:42`
**Problem:** {specific description}
**Suggestion:** {concrete fix}

## Acceptance Criteria Status
- [x] {passed}
- [ ] {failed} ← {why}

Please fix and resubmit.
```
"""
    )

    ### 2.4 Handle Result

    If "APPROVED" in result:
        - Update PLAN.md step:
          - Status: COMPLETED
          - Completed: {current timestamp}
          - Checker Feedback: {summary of approval}
          - Check off acceptance criteria: [x]
        - Update PLAN.md metadata:
          - Current Step: {n+1}
        - Break loop, proceed to next step

    If "NEEDS_WORK" in result:
        - Update PLAN.md step:
          - Attempts: {attempts + 1}
          - Checker Feedback: {feedback from checker}
        - Fix the issues based on feedback
        - Commit fix
        - Continue loop

### 2.5 Escalation (if attempts == 3)

STOP and present to user:

```
🚨 ESCALATION REQUIRED

Step {n}: {title} has failed verification 3 times.

## Feedback History

### Attempt 1:
{checker feedback}

### Attempt 2:
{checker feedback}

### Attempt 3:
{checker feedback}

## Analysis
{What seems to be the recurring issue}

## Options
1. **Provide guidance** - Give hints and I'll continue
2. **Take over** - You implement this step manually
3. **Skip** - Mark skipped and continue to next step
4. **Modify** - Change the step requirements
5. **Abort** - Stop the entire implementation

What would you like me to do?
```

Wait for user response before proceeding.
```

---

## Phase 3: CI & Security Verification (PARALLEL AGENTS)

After all steps are APPROVED, spawn parallel agents for CI and security checks:

### 3.1 Spawn CI & Security Agents (PARALLEL)

Spawn FOUR agents in parallel using a single message with multiple Task tool calls:

```
# PARALLEL AGENT 0: Makefile Verification/Creation
Task(
  subagent_type: "general-purpose",
  description: "Verify/create Makefile",
  prompt: """
You are a build engineer. Verify or create a Makefile for consistent local/CI commands.

## Your Task

### Step 0: Verify Base Project Setup

First, ensure the project has the foundational files:

**0a. Check/create package.json:**
```bash
if [ ! -f package.json ]; then
  npm init -y
  # Update to use ES modules
  npm pkg set type="module"
fi
```

**0b. Check/create tsconfig.json:**
```bash
if [ ! -f tsconfig.json ]; then
  cat > tsconfig.json << 'TSEOF'
{
  "compilerOptions": {
    "target": "ES2022",
    "useDefineForClassFields": true,
    "lib": ["ES2022", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "paths": {
      "@/*": ["./*"]
    }
  },
  "include": ["src", "e2e"]
}
TSEOF
fi
```

**0c. Check/create eslint.config.js:**
```bash
if [ ! -f eslint.config.js ]; then
  cat > eslint.config.js << 'ESLINTEOF'
import js from '@eslint/js';
import tseslint from 'typescript-eslint';
import reactHooks from 'eslint-plugin-react-hooks';
import reactRefresh from 'eslint-plugin-react-refresh';
import globals from 'globals';

export default tseslint.config(
  { ignores: ['dist', 'node_modules'] },
  {
    extends: [js.configs.recommended, ...tseslint.configs.recommended],
    files: ['**/*.{ts,tsx}'],
    languageOptions: {
      ecmaVersion: 2022,
      globals: globals.browser,
    },
    plugins: {
      'react-hooks': reactHooks,
      'react-refresh': reactRefresh,
    },
    rules: {
      ...reactHooks.configs.recommended.rules,
      'react-refresh/only-export-components': ['warn', { allowConstantExport: true }],
      '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
    },
  },
);
ESLINTEOF
fi
```

**0d. Check/create vite.config.ts:**
```bash
if [ ! -f vite.config.ts ]; then
  cat > vite.config.ts << 'VITEEOF'
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': new URL('./', import.meta.url).pathname,
    },
  },
});
VITEEOF
fi
```

**0e. Install base dependencies if missing:**
```bash
# Check if vite is installed
if ! npm ls vite >/dev/null 2>&1; then
  npm install vite @vitejs/plugin-react
  npm install -D typescript @types/react @types/react-dom
fi
```

### Step 1: Check if Makefile exists
```bash
ls -la Makefile
```

2. If Makefile exists, verify it includes these targets:
   - `make dev` - start dev server
   - `make build` - production build
   - `make test` - unit tests
   - `make test-e2e` - E2E tests
   - `make lint` - ESLint
   - `make knip` - dead code detection
   - `make typecheck` - TypeScript check
   - `make check` - all checks (lint, knip, typecheck, test, build)
   - `make ci` - full CI (check + test-e2e)

3. If Makefile is missing or incomplete, create/update it:

```makefile
.PHONY: dev build test test-e2e test-e2e-mobile lint lint-fix knip typecheck check validate ci clean

# Development
dev:
	npm run dev

# Build
build:
	npm run build

# Testing - Unit
test:
	npm run test

# Testing - E2E (Desktop Chromium)
test-e2e:
	npx playwright test --project=chromium

# Testing - E2E (Mobile browsers)
test-e2e-mobile:
	npx playwright test --project="Mobile Chrome" --project="Mobile Safari"

# Testing - E2E (All browsers)
test-e2e-all:
	npx playwright test

# Testing - E2E with UI
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

# Combined Checks (fast, no E2E)
check: lint knip typecheck test build

# Validate Everything (single command for all checks + E2E)
validate: check test-e2e

# CI Pipeline (matches GitHub Actions)
ci: check test-e2e-all

# Cleanup
clean:
	rm -rf dist node_modules/.cache .playwright
```

4. Verify required dev dependencies exist in package.json:

   **Linting & Code Quality:**
   - eslint, @eslint/js, typescript-eslint, globals
   - eslint-plugin-react-hooks, eslint-plugin-react-refresh
   - knip

   **Testing:**
   - @playwright/test
   - vitest, @vitest/coverage-v8, jsdom
   - @testing-library/react, @testing-library/jest-dom

   If missing, add them:
   ```bash
   npm install -D eslint @eslint/js typescript-eslint globals eslint-plugin-react-hooks eslint-plugin-react-refresh knip @playwright/test vitest @vitest/coverage-v8 jsdom @testing-library/react @testing-library/jest-dom
   ```

5. Create/update Playwright config with multi-browser projects (`playwright.config.ts`):

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [
    ['html'],
    ...(process.env.CI ? [['github' as const]] : []),
  ],
  use: {
    baseURL: 'http://localhost:5173',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },

  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
    {
      name: 'Mobile Safari',
      use: { ...devices['iPhone 12'] },
    },
  ],

  webServer: {
    command: 'make dev',
    url: 'http://localhost:5173',
    reuseExistingServer: !process.env.CI,
  },
});
```

6. Create/update vitest.config.ts:
```bash
if [ ! -f vitest.config.ts ]; then
  cat > vitest.config.ts << 'VITESTEOF'
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: ['./src/test/setup.ts'],
    include: ['src/**/*.test.{ts,tsx}'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'html'],
      include: ['src/**/*.{ts,tsx}'],
      exclude: ['src/test/**', '**/*.test.{ts,tsx}'],
    },
  },
});
VITESTEOF
fi
```

7. Create e2e and src directories if missing:
   ```bash
   mkdir -p e2e src/test

   # Create vitest setup file if missing
   if [ ! -f src/test/setup.ts ]; then
     echo "import '@testing-library/jest-dom';" > src/test/setup.ts
   fi
   ```

8. Commit if changes made:
   ```bash
   git add Makefile playwright.config.ts e2e/ package.json package-lock.json
   git commit -m "Add Makefile and Playwright config for consistent build/test"
   ```

## Output
MAKEFILE_READY or MAKEFILE_CREATED with details of what was added.
"""
)
```

Then spawn the remaining three agents:

```
# PARALLEL AGENT 1: CI Workflow Verification/Creation
Task(
  subagent_type: "general-purpose",
  description: "Verify/create CI workflow",
  prompt: """
You are a CI/CD specialist. Verify or create GitHub Actions workflow.

## Your Task

1. Check if workflow exists:
   ```bash
   ls -la .github/workflows/
   ```

2. If workflow exists, verify it includes:
   - Lint step (`make lint`)
   - Knip step (`make knip`) - dead code detection
   - Typecheck step (`make typecheck`)
   - Unit test step (`make test`)
   - Build step (`make build`)
   - E2E test step (`make test-e2e`) - REQUIRED
   - Triggers on PR

3. If workflow is missing or incomplete, create `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  push:
    branches: [main, master]
  pull_request:
    branches: [main, master]

jobs:
  check:
    name: Lint, Knip, Typecheck, Test, Build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - name: Lint
        run: make lint
      - name: Knip (dead code check)
        run: make knip
      - name: Typecheck
        run: make typecheck
      - name: Unit Tests
        run: make test
      - name: Build
        run: make build

  e2e:
    name: E2E Tests (Shard ${{ matrix.shard }}/${{ strategy.job-total }})
    runs-on: ubuntu-latest
    needs: check
    strategy:
      fail-fast: false
      matrix:
        shard: [1, 2, 3]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - name: Install Playwright
        run: npx playwright install --with-deps chromium
      - name: Run E2E Tests (Shard ${{ matrix.shard }}/3)
        run: npx playwright test --project=chromium --shard=${{ matrix.shard }}/3
      - name: Upload E2E Report
        uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-report-shard-${{ matrix.shard }}
          path: playwright-report/
          retention-days: 7

  e2e-mobile:
    name: E2E Mobile Tests
    runs-on: ubuntu-latest
    needs: check
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - name: Install Playwright
        run: npx playwright install --with-deps chromium webkit
      - name: Run Mobile E2E Tests
        run: make test-e2e-mobile
      - name: Upload Mobile E2E Report
        uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-report-mobile
          path: playwright-report/
          retention-days: 7
```

**CI Features:**
- **3 parallel shards** for desktop E2E (3x faster)
- **Separate mobile job** for Mobile Chrome + Mobile Safari
- Uses Makefile targets for consistency between local and CI

Commit if changes made.

## Output
Report: CI_READY or CI_CREATED with details.
"""
)

# PARALLEL AGENT 2: Secrets Scan
Task(
  subagent_type: "general-purpose",
  description: "Scan for secrets",
  prompt: """
You are a security auditor. Scan for accidentally committed secrets.

## Your Task

1. Scan for secret patterns in changes:
   ```bash
   git diff origin/main...HEAD --name-only | xargs -I {} sh -c '
     grep -l -E "(API_KEY|SECRET|PASSWORD|TOKEN|PRIVATE_KEY|aws_access_key|client_secret)" {} 2>/dev/null
   '
   ```

2. Check for .env files:
   ```bash
   git diff origin/main...HEAD --name-only | grep -E "\.env$|\.env\.|credentials|secrets"
   ```

3. Check for hardcoded API keys:
   ```bash
   git diff origin/main...HEAD | grep -E "(sk-[a-zA-Z0-9]{20,}|ghp_[a-zA-Z0-9]{36}|xox[baprs]-[a-zA-Z0-9-]+)"
   ```

4. Verify .gitignore includes:
   - .env, .env.*
   - *.pem, *.key
   - credentials.json, secrets.yaml

## Output
SECRETS_CLEAR or SECRETS_FOUND with list of issues.

**If secrets found, this is a BLOCKING issue.**
"""
)

# PARALLEL AGENT 3: Full Test Suite (using Makefile)
Task(
  subagent_type: "general-purpose",
  description: "Run full test suite",
  prompt: """
Run the complete test suite using Makefile commands.

## Your Task

Run the full CI check locally:

```bash
make ci
```

This runs (in order):
1. `make lint` - ESLint checks
2. `make knip` - Dead code / unused export detection
3. `make typecheck` - TypeScript compiler check
4. `make test` - Vitest unit tests
5. `make build` - Production build
6. `make test-e2e` - Playwright E2E tests

If `make ci` fails, run individual commands to identify the issue:
```bash
make lint      # Check for lint errors
make knip      # Check for dead code
make typecheck # Check for type errors
make test      # Check for test failures
make build     # Check for build errors
make test-e2e  # Check for E2E failures
```

## Output
ALL_PASS or FAILURES with specific command that failed and error details.
"""
)
```

Wait for all four agents to complete.

### 3.2 Handle Results

- **If MAKEFILE missing targets:** Agent should have created/updated it.
- **If SECRETS_FOUND:** STOP immediately. Remove secrets, update .gitignore, re-run.
- **If CI workflow issues:** Fix workflow and commit.
- **If test failures:** Fix and re-run Phase 3.
- **All clear:** Proceed to Phase 4.

---

## Phase 4: Integration

### 4.1 Push and Create PR
```bash
git push -u origin {branch_name}
gh pr create --title "{feature description}" --body "## Summary
- {step 1 summary}
- {step 2 summary}
...

## Test Plan
- [ ] All tests passing
- [ ] Visual verification complete
- [ ] CI checks pass"
```

### 4.2 Monitor CI
```bash
gh pr checks {pr_number}
```

If CI fails, diagnose and fix.

### 4.3 Finalize PLAN.md

Update PLAN.md with completion information:

```markdown
## Metadata
- **Status:** COMPLETED
- **Completed:** {current timestamp}
- **PR:** {pr_url}

## Completion Summary
Feature "{title}" successfully implemented.
- Total steps: {n}
- Total attempts: {sum of all step attempts}
- PR: {url}
- All tests passing
```

### 4.4 Report Completion

```
✅ Feature Implementation Complete

Branch: {branch}
PR: {url}
CI: {status}
Plan: .plans/{feature}/PLAN.md

Steps completed:
1. ✓ {step 1}
2. ✓ {step 2}
...
```

---

## Important Rules

1. **Never skip verification** - Every step must pass the Checker
2. **Checker uses Chrome** - Visual verification is required when applicable
3. **Commit after each step** - Keep changes trackable
4. **Escalate at 3 failures** - Don't spin forever
5. **Independent checking** - Checker has fresh context; trust its assessment
6. **E2E tests are mandatory** - Every new feature needs E2E tests, not just UI features
7. **E2E tests are the real validation** - Unit tests pass but features break; E2E catches real bugs
8. **Use Makefile commands** - `make ci` runs all checks locally; same commands in GitHub Actions
9. **Knip for dead code** - No unused exports or dead code allowed; `make knip` enforces this
10. **Ensure CI exists** - Verify or create GitHub Actions workflow before PR
11. **Scan for secrets** - Never push commits containing credentials or API keys
12. **Verify .gitignore** - Sensitive files must be excluded from version control
13. **Maximize parallelism** - Spawn multiple agents in parallel whenever tasks are independent
14. **Single message, multiple Tasks** - Use one message with multiple Task tool calls for parallel work
