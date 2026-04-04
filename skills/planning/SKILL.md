---
name: planning
description: Use when you have a completed spec at working/spec.md to create an implementation plan.
---

# Planning

You operate as a state machine, dispatching agents and reading files strictly
according to the process flow.

## Iron Law

YOU DO NOT THINK, INTERPRET, SUMMARIZE, OR DECIDE.

NEVER DOUBT THE PROCESS FLOW.

## File Paths

- `working/spec.md` - Spec file
- `working/plan.md` - Plan file
- `working/plan-review-results.md` - Review results

## Agent Prompt Format

Use EXACT format only. Any extra content = violation.

```
# planner
- Spec path: working/spec.md
- Plan path: working/plan.md
- Review results path: working/plan-review-results.md

# plan-reviewer
- Spec path: working/spec.md
- Plan path: working/plan.md
- Review results path: working/plan-review-results.md
```

## Process Flow

```
Setup:
    read `spec.md` → output summary
    wait user confirm → begin Step 1

Step 1: dispatch planner
Step 2: dispatch plan-reviewer
Step 3: read issues in `plan-review-results.md`
    → has `Pending` issues: go to Step 1 (fix issues and do reviewing again)
    → no `Pending` issues: complete

output the dispatch count, tokens and duration for each agent
```

**DO NOT:**
- Skip any step of process flow
- Combine steps of process flow
- Reorder steps of process flow (Plan → Plan review, always)
- Stop iterating because "taking too long"
- Decide plan is "good enough" yourself
- Fix, verify or review the plan yourself — dispatch the corresponding agent
- Add context/explanations to agent prompts
- Interpret/summarize agent response
