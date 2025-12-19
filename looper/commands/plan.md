# /plan - Create Implementation Plan

Create a structured implementation plan for a feature without implementing it.

## Arguments
- $ARGUMENTS: The feature description

---

## Setup

First, create the plan directory:

```bash
# Derive feature slug from description
FEATURE_SLUG=$(echo "$ARGUMENTS" | tr ' ' '-' | tr '[:upper:]' '[:lower:]' | cut -c1-50)

# Create plan directory
mkdir -p .plans/${FEATURE_SLUG}
```

Plan will be created at: `.plans/{feature-slug}/PLAN.md`

---

## Your Task

### 1. Explore the Codebase

Before planning, understand the project:

```
- Use Glob to find relevant files by pattern
- Use Grep to search for related code
- Use Read to examine key files
```

Identify:
- Project structure and tech stack
- Existing patterns and conventions
- Related files and dependencies
- Existing tests to maintain

### 2. Break Into Steps

Create 2-4 discrete implementation steps where:
- Each step is independently verifiable
- Each step can be committed separately
- Steps are ordered by dependency (do X before Y)
- Each step is 15-60 minutes of work

### 3. Define Acceptance Criteria

For each step, define:
- Specific, verifiable criteria (not vague)
- What should be visually checkable in browser (if applicable)
- What tests should pass

### 4. Plan Test Coverage (E2E PRIORITY)

For each step, specify:
- **E2E tests (REQUIRED):** What user flows need E2E tests - this is how you actually validate the feature works. E2E tests are mandatory for ALL features, not just visual ones.
- **Integration tests:** What component/service connections need testing
- **Unit tests:** Only for complex algorithms or utility functions that benefit from isolated testing
- **Test file locations:** Where tests should be created (e.g., `e2e/*.spec.ts`)

**Philosophy:** Unit tests pass but features break. E2E tests simulate real user behavior and catch the bugs that matter.

---

## Output: Create PLAN.md

Create the plan at `.plans/{feature-slug}/PLAN.md` with this exact structure:

```markdown
# Implementation Plan: {Feature Title}

## Metadata
- **Status:** NOT_STARTED
- **Current Step:** 1
- **Created:** {YYYY-MM-DDTHH:MM:SS}
- **Completed:** -
- **Branch:** feature/{slug}
- **PR:** -

## Overview

{2-3 sentence description of what this feature accomplishes}

## Tech Stack Context

- **Framework:** {React, Next.js, Vue, etc.}
- **Key Directories:** {relevant paths}
- **Patterns to Follow:** {existing patterns to match}

## Steps

### Step 1: {Short Descriptive Title}
- **Status:** NOT_STARTED
- **Attempts:** 0
- **Started:** -
- **Completed:** -

**Description:**
{Detailed description of what to implement. Be specific about what to create, modify, or connect.}

**Files to Change:**
| File | Action | Details |
|------|--------|---------|
| `src/components/Thing.tsx` | Create | New component for X |
| `src/pages/index.tsx` | Modify | Import and use Thing |

**Acceptance Criteria:**
- [ ] {Specific verifiable criterion}
- [ ] {Specific verifiable criterion}
- [ ] {Specific verifiable criterion}

**Visual Check:**
Navigate to `{url}` and verify:
- {What should appear}
- {What interaction should work}
(Or "N/A - no visual component" if purely backend)

**Tests to Write (E2E Required):**
| Type | File | Description |
|------|------|-------------|
| E2E | `e2e/thing.spec.ts` | Test user flow: {describe what user does} |
| E2E | `e2e/thing-edge-cases.spec.ts` | Test edge cases: {error states, boundaries} |
| Unit | `src/__tests__/thing.test.ts` | (Optional) Complex algorithm tests |

**Test Command:** `{test command}`

**Checker Feedback:** -

---

### Step 2: {Short Descriptive Title}
- **Status:** NOT_STARTED
- **Attempts:** 0
- **Started:** -
- **Completed:** -

**Description:**
{...}

**Files to Change:**
| File | Action | Details |
|------|--------|---------|

**Acceptance Criteria:**
- [ ] {criterion}

**Visual Check:**
{...}

**Tests to Write (E2E Required):**
| Type | File | Description |
|------|------|-------------|
| E2E | `e2e/...` | Test user flow: ... |

**Test Command:** `{test command}`

**Checker Feedback:** -

---

{Continue for remaining steps...}

## Risks and Considerations

- {Edge cases to watch for}
- {Potential breaking changes}
- {Dependencies on external systems}

## Completion Summary
-
```

---

## Guidelines

### Do:
- **Be specific**: "Add submit button in LoginForm that calls handleSubmit on click" ✓
- **Keep steps small**: Each should be independently completable
- **Define visual checks**: The Checker will use Chrome MCP to verify
- **Order by dependency**: Can't use a component before creating it

### Don't:
- **Be vague**: "Add a button" ✗
- **Over-scope**: If you need 5+ steps, the feature should be split
- **Skip exploration**: Always understand the codebase first
- **Implement anything**: This command only creates the plan
