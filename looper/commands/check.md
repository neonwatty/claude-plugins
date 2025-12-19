# /check - Verify Current Implementation

Spawn an independent Checker agent to verify the current step's implementation.

## Arguments
- $ARGUMENTS: Optional - step number or path to PLAN.md
  - `/check` - Check current step using PLAN.md in current feature's .plans/ directory
  - `/check 2` - Check step 2 specifically
  - `/check .plans/my-feature/PLAN.md` - Check using specific plan file

---

## Prerequisites

Before running this command:
- PLAN.md must exist in `.plans/{feature}/` directory
- Changes should be committed
- Dev server should be running (if visual check required)

---

## Process

### 1. Read Current State

```bash
# Get current step from PLAN.md
# Get step details: description, acceptance criteria, visual check

# Get recent changes
git diff HEAD~1

# Run tests
{test command}
```

### 2. Spawn Checker Agent

Use Task tool:

```
Task(
  subagent_type: "general-purpose",
  description: "Verify step implementation",
  prompt: """
You are an independent QA checker. Verify the implementation is functionally correct. Be objective.

## Step Being Verified
**Title:** {step_title}
**Description:** {step_description}
**Acceptance Criteria:** {criteria}
**Visual Check:** {visual_check_description}

## Changes Made
```diff
{git diff output}
```

## Test Results
```
{test output}
```

## App URL
{app_url}

---

## Your Verification Tasks

### 1. Code Review (Functional Focus)

Read the changed files and verify:
- [ ] Logic correctly implements what's described
- [ ] No obvious bugs or broken logic
- [ ] Edge cases handled appropriately

Do NOT nitpick style, formatting, or naming conventions.

### 2. Visual Verification

If Visual Check is not "N/A", you MUST use Chrome MCP:

**Step 2a: Navigate**
```
mcp__claude-in-chrome__navigate
url: {app_url}
```

**Step 2b: Screenshot**
```
mcp__claude-in-chrome__computer
action: screenshot
```

**Step 2c: Read Page Structure**
```
mcp__claude-in-chrome__read_page
```

**Step 2d: Find Specific Elements**
```
mcp__claude-in-chrome__find
query: {element from visual check criteria}
```

**Step 2e: Test Interactions (if applicable)**
```
mcp__claude-in-chrome__computer
action: left_click
coordinate: [x, y]
```

**Step 2f: Verify State Changes**
```
mcp__claude-in-chrome__javascript_tool
text: {js to check state}
```

### 3. Make Decision

**APPROVE if:**
- All acceptance criteria verifiably met
- Tests pass
- Visual check confirms expected UI
- No functional bugs found

**REJECT if:**
- Any criterion not met
- Tests fail
- Visual check shows incorrect UI
- Functional bug discovered

---

## Output Format

### If Approved:

```
APPROVED

## Verification Summary

### Code Review
✓ {what you verified in code}
✓ {what you verified in code}

### Visual Check
✓ {what you saw and confirmed}
Screenshot confirms: {description of what's visible}

### Functional Check
✓ {interaction tested and result}

All acceptance criteria met.
```

### If Needs Work:

```
NEEDS_WORK

## Issues Found

### Issue 1: {Short title}
**Location:** `path/to/file.ts:42`
**Problem:** {specific description of what's wrong}
**Suggestion:** {concrete fix recommendation}

### Issue 2: {Short title}
**Location:** {where}
**Problem:** {what's wrong}
**Suggestion:** {how to fix}

## Acceptance Criteria Status
- [x] {criterion that passed}
- [ ] {criterion that failed} ← {why it failed}

## Visual Check Result
{What was wrong visually, with screenshot reference}

---
Please fix the above issues and resubmit.
```
"""
)
```

### 3. Report Result

**If APPROVED:**
- Update PLAN.md to mark step as APPROVED
- Report success to user

**If NEEDS_WORK:**
- Display issues found
- Suggest specific fixes
- Do not update PLAN.md status

---

## Usage Examples

```
/check          # Check current step from PLAN.md
/check 2        # Check step 2 specifically
/check 3        # Check step 3 specifically
```

---

## Important Notes

1. **Visual verification is required** when Visual Check is defined
2. **Checker has fresh context** - it doesn't know what was "hard" to implement
3. **Be specific in feedback** - vague issues are useless
4. **Binary decision** - either APPROVED or NEEDS_WORK, no middle ground
