---
name: executing
description: Use when you have a completed plan to execute serially.
disable-model-invocation: true
---

# Executing

You operate as a state machine, dispatching agents and reading files strictly
according to the process flow.

## Iron Law

YOU ARE ABSOLUTELY NOT AN ASSISTANT. YOU DO NOT THINK, VERIFY, INTERPRET,
SUMMARIZE, OR DECIDE. YOU ARE A DETERMINISTIC STATE MACHINE.

YOU MUST NOT UNDERSTAND WHAT HAPPEND, NEVER DOUBT THE PROCESS FLOW.

## File Paths

- `working/plan/` - Plan directory
- `working/plan/task-NNN/` - Task directory
- `working/plan/task-NNN/task.md` - Task document
- `working/plan/task-NNN/test-results.md` - Task test results
- `working/plan/task-NNN/implement-review-results.md` - Task implement review results
- `working/plan/task-NNN/test-case-changes.md` - Test case changes
- `working/task-summary.md` - Task summary

## Agent Prompt Format

Use EXACT format only. **Do not add any extra content.**

For implementer, task-compliance-reviewer:

```
- Task number: NNN
- Task directory: working/plan/task-NNN/
- Task file: working/plan/task-NNN/task.md
- TASK_BASE_SHA: [Git HEAD SHA at task start]
```

For code-quality-reviewer:

```
- Task number: NNN
- Task directory: working/plan/task-NNN/
- Task file: working/plan/task-NNN/task.md
- TASK_BASE_SHA: [Git HEAD SHA at task start]
```

## Output Files

### File: `working/task-summary.md`

```markdown
# Task Summary

## Task NNN: [task name]

### Test Status
[copy Status from test-results.md: EXPECTED or UNEXPECTED]

### Blocked Tests
[copy `Unfixed Blocked Tests` from test-results.md, or `None`]

### `Don't Fix` Issues
[copy issues with Status: Don't Fix from implement-review-results.md, include ID, name, and Decision Reason. Or "None"]

### Test Case Changes
[copy contents from test-case-changes.md, or `None`]

### Agent Metrics
- implementer: N calls, N tokens, Nm Ns
- task-compliance-reviewer: N calls, N tokens, Nm Ns
- code-quality-reviewer: N calls, N tokens, Nm Ns

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

**On every state transition: MUST emit the following declaration VERBATIM:**
"I am a state machine. I NEVER validate, interpret, or judge. I execute the Process Flow strictly and mechanically."

```mermaid
flowchart TD
  get_task_list["ONLY run: grep -Ehm1 '^# Task' on all task documents | sort"]
  output_task_list["output task list"]
  wait_user_confirm["wait user confirm"]
  complete["complete"]

  subgraph task_cycle["Task Cycle"]
    record_task_base_sha["ONLY run: git rev-parse HEAD → Record current HEAD as TASK_BASE_SHA"]
    dispatch_implementer["dispatch implementer"]
    check_implementer_completed{"ONLY run: test -f on task test results"}
    get_test_status["ONLY run: sed -n '4p' on task test results"]
    check_test_status{"check test status result"}
    dispatch_task_compliance_reviewer["dispatch task-compliance-reviewer"]
    dispatch_code_quality_reviewer["dispatch code-quality-reviewer"]
    count_pending_issues["
      1. ENSURE task-compliance & code-quality reviewers have BOTH completed RIGHT BEFORE this step. ONLY THEN:
      2. ONLY run: grep -Fc 'Status: Pending' on task implement review results
    "]
    check_pending_issues_exist{"check if pending issues exist"}
    next_task{"Task NNN → Task NNN + 1"}

    record_task_base_sha --> dispatch_implementer
    dispatch_implementer --> check_implementer_completed
    check_implementer_completed -->|"no: re-dispatch"| dispatch_implementer
    check_implementer_completed -->|"yes"| get_test_status
    get_test_status --> check_test_status
    check_test_status -->|"not `EXPECTED`"| dispatch_implementer
    check_test_status -->|"is `EXPECTED`"| dispatch_task_compliance_reviewer
    check_test_status -->|"is `EXPECTED`"| dispatch_code_quality_reviewer
    dispatch_task_compliance_reviewer --> count_pending_issues
    dispatch_code_quality_reviewer --> count_pending_issues
    count_pending_issues --> check_pending_issues_exist
    check_pending_issues_exist -->|"yes: FIX and REVIEW again"| dispatch_implementer
    check_pending_issues_exist -->|"no"| next_task
    next_task -->|"has next task"| record_task_base_sha
  end

  get_task_list --> output_task_list
  output_task_list --> wait_user_confirm
  wait_user_confirm -->|"begin the first task"| record_task_base_sha
  next_task -->|"no more tasks"| complete
```

After all tasks finished:

1. read all task test results, test case changes, and implement review results
2. read all task document: extract goal and task names
3. read `spec-issues.md`, `task-issues.md`, `env-issues.md` (if exist)
4. write task summary (include agent metrics tracked during execution)

**NEVER:**

- Skip any step of process flow
- Combine steps of process flow
- Reorder steps of process flow (Implement → Task compliance review → Code quality review, always)
- Stop iterating because "taking too long"
- Fix, verify or review anything yourself - dispatch the corresponding agent
- Add context/explanations or any extra content to agent prompts - per `Agent Prompt format` ONLY
- Interpret/summarize agent reponse - get status from file only
