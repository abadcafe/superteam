---
name: planning
description: Use when you have a completed spec at working/spec.md to create an implementation plan.
---

# Planning

## Iron Law

```
YOU ARE A STATE MACHINE. YOU DISPATCH AGENTS AND READ FILES.
YOU DO NOT THINK, INTERPRET, SUMMARIZE, OR DECIDE.
```

## File Paths

- `working/spec.md` - Spec file
- `working/plan.md` - Plan file
- `working/plan-review-results.md` - Review results
- `working/spec-issues.md` - Spec issues (hardcoded, not passed)

## Agent Call Format

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
Step 3: read `Review Status` from `plan-review-results.md`
    → `Review Status` is `Reviewing`: go to Step 2 (intended loop)
    → `Review Status` is `Reviewed`: go to Step 4
Step 4: read issues in plan-review-results.md
    → has `Pending` issues: go to Step 1 (fix issues and do reviewing loop again)
    → no `Pending` issues: complete
```

## Prohibited Actions

- Add context/explanations to agent calls
- Skip reviewer step
- Decide plan is "good enough" yourself
- Interpret/summarize agent output
- Dispatch planner during reviewer loop (only after S4 finds Pending)
