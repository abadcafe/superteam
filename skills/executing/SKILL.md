---
name: executing
description: Use when you have a completed plan at working/plan.md to execute serially.
---

# Executing Plans

## Iron Law

```
YOU ARE A STATE MACHINE. YOU DISPATCH AGENTS AND READ FILES.
YOU DO NOT THINK, INTERPRET, SUMMARIZE, OR DECIDE.
```

## File Paths

- `working/plan.md` - Plan file
- `working/artifacts/task-NNN/` - Task output directory
- `working/plan-issues.md` - Plan issues (hardcoded)
- `working/env-issues.md` - Environment issues (hardcoded)

## Agent Call Format

Use EXACT format only. Any extra content = violation.

```
# environment-checker
- Plan path: working/plan.md

# implementer
- Plan path: working/plan.md
- Task number: NNN
- Task output directory: working/artifacts/task-NNN/

# spec-reviewer
- Plan path: working/plan.md
- Task number: NNN
- Task output directory: working/artifacts/task-NNN/

# code-reviewer
- Plan path: working/plan.md
- Task number: NNN
- Task output directory: working/artifacts/task-NNN/

# tester
- Plan path: working/plan.md
- Task number: NNN
- Task output directory: working/artifacts/task-NNN/
```

## Process Flow

```
Setup:
    read `plan.md` → output summary
    dispatch environment-checker
    read `env-issues.md` → output to user
    wait user confirm → begin Task 001

Per-Task (NNN = 001, 002, ...):
Step 1: dispatch implementer
    read `Status` of `whitebox-test-results.md`
    → `EXPECTED`: go to Step 2
    → `UNEXPECTED`: go to Step 1 (intended loop)

Step 2: dispatch spec-reviewer
    read `Review Status` in `Spec Review Issues` section of `implement-review-results.md`
    → `Review Status` is `Reviewing`: go to Step 2 (intended loop)
    → `Review Status` is `Reviewed`: go to Step 3

Step 3: dispatch code-reviewer
    read `Review Status` in `Code Review Issues` section of `implement-review-results.md`
    → `Review Status` is `Reviewing`: go to Step 3 (intended loop)
    → `Review Status` is `Reviewed`: go to Step 4

Step 4: dispatch tester

Step 5: read `implement-review-results.md`
    → has `Pending` issues: go to Step 1 (intended loop: Step 1 → 2 → 3 → 4 → 5 again)
    → no `Pending` issues: next task

After all tasks: complete
```

## Prohibited Actions

- Add context/explanations to agent calls
- Skip any step/reviewer
- Reorder steps (Spec → Code → Tester, always)
- Combine tasks into one dispatch
- Fix code yourself — dispatch Implementer
- Interpret/summarize agent output — read status only
- Decide issue "not worth fixing" — Implementer's job
- Stop iterating because "taking too long"
- Dispatch Implementer during reviewer loops — only after Step 5 finds Pending
- Make decisions not covered by steps — STOP and wait for human
