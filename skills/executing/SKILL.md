---
name: executing
description: Use when you have a completed plan at working/plan.md to execute serially.
---

# Executing

You operate as a state machine, dispatching agents and reading files strictly
according to the process flow.

## Iron Law

YOU DO NOT THINK, INTERPRET, SUMMARIZE, OR DECIDE.

NEVER DOUBT THE PROCESS FLOW.

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
    wait user confirm → begin Task 001

Per-Task (NNN = 001, 002, ...):
Step 1: dispatch implementer
    read `Status` of `test-results.md`
    → `UNEXPECTED`: go to Step 1 (intended loop for fix `UNEXPECTED` cases)
    → `EXPECTED`: go to Step 2
Step 2: dispatch spec-reviewer
Step 3: dispatch code-reviewer
Step 5: read `implement-review-results.md`
    → has `Pending` issues: go to Step 1 (intended loop: Step 1 → 2 → 3 → 4 → 5 again for fix `Pending` issues)
    → no `Pending` issues: next task

After all tasks: complete
```

## Do not

- Skip any step of process flow
- Combine steps of process flow
- Reorder steps of process flow (Implement → Spec review → Code review, always)
- Combine tasks into one dispatch
- Stop iterating because "taking too long"
- Decide issue "not worth fixing" — Implementer's job
- Fix code yourself — dispatch Implementer
- Add context/explanations to agent calls
- Interpret/summarize agent output — read status only
- Make decisions not covered by steps — STOP and wait for human
