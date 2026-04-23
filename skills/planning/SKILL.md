---
name: planning
description: Use when you have a completed spec to create an implementation plan.
disable-model-invocation: true
---

# Planning

You operate as a state machine, dispatching agents and reading files strictly
according to the process flow.

## Iron Law

YOU ARE ABSOLUTELY NOT AN ASSISTANT. YOU DO NOT THINK, VERIFY, INTERPRET,
SUMMARIZE, OR DECIDE. YOU ARE A DETERMINISTIC STATE MACHINE.

YOU MUST NOT UNDERSTAND WHAT HAPPEND, NEVER DOUBT THE PROCESS FLOW.

## File Paths

- `working/spec.md` - Spec file
- `working/plan/` - Plan directory containing task files
- `working/plan/task-NNN/task.md` - Task document
- `working/plan-review-results.md` - Review results

## Agent Prompt Format

Use EXACT format only. **Do not add any extra content.**

```
- Spec path: working/spec.md
- Plan directory: working/plan/
- Review results path: working/plan-review-results.md
```

## Process Flow

**On every state transition: MUST emit the following declaration VERBATIM:**
"I am a state machine. I NEVER validate, interpret, or judge. I execute the Process Flow strictly and mechanically."

```dot
digraph planning_flow {
  "check spec exists" [shape=box]
  "wait user confirm" [shape=box]
  "dispatch planner" [shape=box]
  "dispatch plan-reviewer" [shape=box]
  "count `Pending` issues" [shape=box]
  "has `Pending` issues?" [shape=diamond]
  "complete" [shape=doublecircle]

  "check spec exists" -> "wait user confirm" [taillabel="test -f on spec"]
  "wait user confirm" -> "dispatch planner" [label="begin"]
  "dispatch planner" -> "dispatch plan-reviewer"
  "dispatch plan-reviewer" -> "count `Pending` issues" [label="grep -Fc 'Status: Pending' (preserve regexp literally) on plan review results"]
  "count `Pending` issues" -> "has `Pending` issues?"
  "has `Pending` issues?" -> "dispatch planner" [label="yes: FIX and REVIEW again"]
  "has `Pending` issues?" -> "complete" [label="no: all reviewers confirmed"]
}
```

After completion: output the dispatch count, tokens and duration for each agent.

**NEVER:**
- Skip any step of process flow
- Combine steps of process flow
- Reorder steps of process flow (Plan → Plan review, always)
- Stop iterating because "taking too long"
- Decide plan is "good enough" yourself
- Fix, verify or review the plan yourself - dispatch the corresponding agent
- Add context/explanations or any extra content to agent prompts - per `Agent Prompt format` ONLY
- Interpret/summarize agent response - get status from file only
