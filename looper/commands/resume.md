# /resume - Resume Implementation

Resume an in-progress feature implementation from where it left off.

## Arguments
- $ARGUMENTS: Optional - feature name or path to PLAN.md
  - `/resume` - Auto-detect from current branch (feature/X → .plans/X/PLAN.md)
  - `/resume my-feature` - Resume .plans/my-feature/PLAN.md
  - `/resume .plans/my-feature/PLAN.md` - Resume from specific path

---

## Process

### 1. Load State

Read PLAN.md to determine:
- Feature being implemented
- Current step number
- Step status (NOT_STARTED, IN_PROGRESS, APPROVED)
- Number of attempts on current step

### 2. Check Git State

```bash
git branch --show-current    # Confirm on feature branch
git status                   # Check for uncommitted changes
```

### 3. Report Status

Before resuming, display:

```
📋 Implementation Status

Feature: {plan title}
Branch: {current git branch}
Progress: Step {current} of {total}

Step Status:
  1. {title} - ✅ APPROVED
  2. {title} - 🔄 IN_PROGRESS (attempt {n}/3)
  3. {title} - ⏳ NOT_STARTED
  4. {title} - ⏳ NOT_STARTED

Current Step: {step title}
Attempts: {n}/3

Uncommitted changes: {yes/no}
```

### 4. Resume Execution

Based on current state:

**If current step is IN_PROGRESS:**
- Continue implementation
- Address any previous checker feedback if available
- Follow the implementation loop from `/implement`

**If current step is NOT_STARTED:**
- Begin implementation of this step
- Follow the implementation loop from `/implement`

**If all steps are APPROVED:**
- Proceed to Phase 3 (Integration) from `/implement`
- Create PR if not already created
- Monitor CI

**If there are uncommitted changes:**
- Ask user: commit these changes or discard?
- Proceed based on response

### 5. Continue Loop

Follow the same verification loop as `/implement`:

```
while attempts < 3:
    1. Implement/fix the step
    2. Commit changes
    3. Run tests
    4. Spawn Checker (via /check)
    5. If APPROVED: next step
       If NEEDS_WORK: attempts++, fix issues

if attempts == 3:
    Escalate to user
```

---

## Usage

```
/resume                              # Auto-detect from branch name
/resume my-feature                   # Resume .plans/my-feature/PLAN.md
/resume .plans/other/PLAN.md         # Resume from specific path
```

---

## Recovery Scenarios

### Scenario: Claude Code session ended mid-step

1. `/resume` loads PLAN.md
2. Sees step marked IN_PROGRESS
3. Checks git log for recent commits
4. Continues from where it left off

### Scenario: Tests were failing

1. `/resume` loads state
2. Runs tests to confirm current state
3. If still failing, continues fixing
4. If passing, proceeds to Checker

### Scenario: Checker had rejected, session ended

1. `/resume` loads state
2. Sees previous NEEDS_WORK feedback in attempt history
3. Addresses those specific issues
4. Re-runs Checker

### Scenario: Was in escalation state

1. `/resume` loads state
2. Sees attempts = 3
3. Re-presents escalation options to user
4. Waits for decision

---

## Notes

- Always verify git branch before making changes
- If PLAN.md doesn't exist, suggest running `/plan` first
- If uncommitted changes exist, resolve before proceeding
