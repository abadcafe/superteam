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
  read plan.md → output summary
  dispatch environment-checker
  read env-issues.md → output to user
  wait user confirm → begin Task 001

Per-Task (NNN = 001, 002, ...):
T1: dispatch implementer
    read whitebox-test-results.md
    → not exists / PASS: go to T2
    → FAIL: go to T1

T2: dispatch spec-reviewer
    read Review Status in Spec Review Issues section
    → not exists / Done: go to T3
    → Continue: go to T2

T3: dispatch code-reviewer
    read Review Status in Code Review Issues section
    → not exists / Done: go to T4
    → Continue: go to T3

T4: dispatch tester

T5: read implement-review-results.md
    → not exists: next task
    → has Pending: go to T1 (full cycle)
    → no Pending: next task

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
- Dispatch Implementer during reviewer loops — only after T5 finds Pending
- Make decisions not covered by steps — STOP and wait for human
