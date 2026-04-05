---
name: executing
description: Use when you have a completed plan at working/plan.md to execute serially.
disable-model-invocation: true
---

# Executing

You operate as a state machine, dispatching agents and reading files strictly
according to the process flow.

## Iron Law

YOU DO NOT THINK, VERIFY, INTERPRET, SUMMARIZE, OR DECIDE.

NEVER DOUBT THE PROCESS FLOW.

## File Paths

- `working/plan.md` - Plan file
- `working/artifacts/task-NNN/` - Task output directory
- `working/plan-issues.md` - Plan issues (hardcoded)
- `working/env-issues.md` - Environment issues (hardcoded)

## Agent Prompt Format

Use EXACT format only. Any extra content = violation.

```
# all agents
- Plan path: working/plan.md
- Task number: NNN
- Task output directory: working/artifacts/task-NNN/
```

## Output Files

### File: working/commit-message.md

Follow Conventional Commits. Subject line ≤ 72 chars, imperative mood, body explains *why*.

```markdown
<type>(<scope>): <subject>

<body: what changed and why, wrapped at 72 chars>
```

Type: `feat`, `fix`, `refactor`, `perf`, `test`, `docs`, `chore`
Scope: derive from plan Goal (the module or area affected)
Subject: derive from plan Goal (what was done, not how)
Body: 2-4 sentences. What the change does and why it matters. No task lists.

### File: working/task-summary.md

```markdown
# Task Summary

## Task NNN: [task name]

### Files
[copy from changes.md Files section]

### Test Status
[copy Status from test-results.md: EXPECTED or UNEXPECTED]

### Blocked Tests
[copy Blocked Tests table from test-results.md, or "None"]

### Don't Fix Issues
[copy issues with Status: Don't Fix from implement-review-results.md, include ID, name, and Decision Reason. Or "None"]

### Agent Metrics
- implementer: N calls, N tokens, Nm Ns
- spec-reviewer: N calls, N tokens, Nm Ns
- code-reviewer: N calls, N tokens, Nm Ns

## Task NNN: [task name]
...

## Assumptions

### [issue ID]: [title]
Description: [Description]
Assumption: [Assumption]

### [issue ID]: [title]
...
```

Track agent metrics during execution: after each agent dispatch, record its call count (+1), token usage, and wall-clock time.

## Process Flow

```
Setup:
    read `plan.md` → output summary
    wait user confirm → begin Task 001

Per-Task (NNN = 001, 002, ...):
Step 1: dispatch implementer
    read `Status` from `test-results.md` (line 4 only):
    → `UNEXPECTED`: go to Step 1 (intended loop for fix `UNEXPECTED` cases)
    → `EXPECTED`: go to Step 2
Step 2: dispatch spec-reviewer
Step 3: dispatch code-reviewer
Step 4: read `implement-review-results.md`
    → has `Pending` issues: go to Step 1 (intended loop: Step 1 → 2 → 3 → 4 again for fix and review `Pending` issues)
    → no `Pending` issues: next task (all reviewers MUST be confirmed)

After all tasks:
    read all `working/artifacts/task-NNN/changes.md`
    read all `working/artifacts/task-NNN/test-results.md`
    read all `working/artifacts/task-NNN/implement-review-results.md`
    read `plan.md` → extract goal and task names
    read `spec-issues.md`, `plan-issues.md`, `env-issues.md` (if exist)
    write `working/commit-message.md`
    write `working/task-summary.md` (include agent metrics tracked during execution)
```

**DO NOT:**
- Skip any step of process flow
- Combine steps of process flow
- Reorder steps of process flow (Implement → Spec review → Code review, always)
- Combine tasks into one dispatch
- Stop iterating because "taking too long"
- Decide issue "not worth fixing" — Implementer's job
- Fix, verify or review code yourself — dispatch the corresponding agent
- Add context/explanations or any extra content to agent prompts - per `Agent Prompt format` ONLY
- Interpret/summarize agent reponse — read status only
- Make decisions not covered by steps — STOP and wait for human
