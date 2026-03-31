---
name: verification-before-completion
description: Use when about to write PASS/FAIL status or claim any work is complete.
---

# Verification Before Completion

## Core Principle

```
NO STATUS WRITTEN WITHOUT FRESH VERIFICATION EVIDENCE
```

**Violating the letter of this rule is violating the spirit of this rule.**

## Gate Function

```
BEFORE writing status:
  IDENTIFY: What command proves this claim?
  RUN: Execute FULL command (fresh, complete)
  READ: Full output, check exit code, count failures
  VERIFY: Output confirms claim?
    → NO: state actual status with evidence
    → YES: write status with evidence
  ONLY THEN: write the status
```

Skip any step = lying, not verifying.

## Required Evidence

| Claim | Requires | Not Sufficient |
|-------|----------|----------------|
| Tests pass (PASS) | Test command output: 0 failures | Previous run, "should pass" |
| Build succeeds | Build command: exit 0 | Linter passing, logs look good |
| Bug fixed | Test original symptom: passes | Code changed, assumed fixed |
| Regression test works | Red-green cycle verified | Test passes once |

## Key Patterns

```
White-box tests:
  OK  → [Run test] → [See: 5/5 pass] → Write PASS
  BAD → "Should pass now" → Write PASS

Regression tests (TDD Red-Green):
  OK  → Write test → Run (FAIL) → Implement → Run (PASS)
  BAD → "Written regression test" (no red-green)

Build:
  OK  → [Run build] → [See: exit 0] → "Build passes"
  BAD → "Linter passed" (≠ compilation)
```

## When To Apply

ALWAYS before:
- Writing PASS/FAIL to whitebox-test-results.md
- ANY expression of satisfaction about work state
- ANY positive statement about correctness
- Finishing self-review

## Red Flags - STOP

- Using "should", "probably", "seems to"
- Expressing satisfaction before verification
- About to write PASS without running tests
- Relying on partial verification
- Thinking "just this once"
- ANY wording implying success without verification

**If you catch yourself:** STOP, run the verification.

## Rationalization Prevention

| Excuse | Reality |
|--------|---------|
| "Should work now" | RUN the verification |
| "I'm confident" | Confidence ≠ evidence |
| "Just this once" | No exceptions |
| "Linter passed" | Linter ≠ tests |
| "Partial check is enough" | Partial proves nothing |
| "Different words so rule doesn't apply" | Spirit over letter |