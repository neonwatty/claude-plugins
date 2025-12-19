# /implement - Autonomous Feature Implementation

Implement a feature with independent verification at each step.

## Arguments
- $ARGUMENTS: The feature description

---

## Configuration

Before starting, confirm or ask for:
- **Test Command:** (e.g., `npm test`, `pytest`, `go test ./...`)
- **App URL:** (e.g., `http://localhost:3000`) - for visual verification
- **Dev Server Command:** (e.g., `npm run dev`) - if app needs to be running

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

### 1.2 Spawn Planner Agent

Use the Task tool to spawn a Planner agent:

```
Task(
  subagent_type: "general-purpose",
  description: "Plan feature implementation",
  prompt: """
You are a planning agent. Analyze the feature request and create a step-by-step implementation plan. Do NOT implement anything.

## Feature Request
{$ARGUMENTS}

## Plan Location
Create the plan at: .plans/{feature_slug}/PLAN.md

## Your Task

1. **Explore the codebase** to understand:
   - Project structure and tech stack
   - Existing patterns and conventions
   - Related files and dependencies

2. **Break down** the feature into 2-4 discrete steps where:
   - Each step is independently verifiable
   - Each step can be committed separately
   - Steps are ordered by dependency

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

### 1.3 Review Plan

After Planner creates PLAN.md, review it before proceeding. Confirm the steps make sense.

---

## Phase 2: Step Execution

For each step in PLAN.md, execute this loop:

```
attempts = 0
max_attempts = 3

while attempts < max_attempts:

    ### 2.1 Implement
    - Read step description from PLAN.md
    - Implement the changes
    - Commit: git commit -m "Step {n}: {title}"

    ### 2.2 Run Tests
    - Execute test command
    - If tests fail: fix issues, attempts++, continue loop

    ### 2.3 Spawn Checker Agent

    Use Task tool to spawn Checker:

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

### 2. Visual Verification (Required when Visual Check is not "N/A")
Use Chrome MCP tools:

1. Navigate: mcp__claude-in-chrome__navigate to App URL
2. Screenshot: mcp__claude-in-chrome__computer with action: screenshot
3. Read page: mcp__claude-in-chrome__read_page
4. Find elements: mcp__claude-in-chrome__find for specific elements
5. Test interactions: mcp__claude-in-chrome__computer with action: left_click
6. Run JS checks: mcp__claude-in-chrome__javascript_tool if needed

### 3. Decision

**APPROVE if:**
- All acceptance criteria met
- Tests pass
- Visual check confirms expected UI (if applicable)
- No functional bugs

**REJECT if:**
- Any criterion not met
- Tests fail
- Visual shows incorrect UI
- Functional bug found

---

## Output Format

Respond with EXACTLY one of:

### If Approved:
```
APPROVED

## Verification Summary
### Code Review
✓ {what you verified}

### Visual Check
✓ {what you saw}
Screenshot confirms: {description}

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

## Phase 3: Integration

After all steps are APPROVED:

### 3.1 Final Verification
- Run full test suite
- Ensure all changes are committed

### 3.2 Push and Create PR
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

### 3.3 Monitor CI
```bash
gh pr checks {pr_number}
```

If CI fails, diagnose and fix.

### 3.4 Finalize PLAN.md

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

### 3.5 Report Completion

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
