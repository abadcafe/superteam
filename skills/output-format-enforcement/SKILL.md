---
name: output-format-enforcement
description: Use when starting any agent execution to enforce exact output file formats.
---

# Output Format Enforcement

Invoke BEFORE any other work.

## Core Principles

1. **Exact headers** — Skill matches string literals. Wrong header = file not recognized.
2. **Exact status values** — PASS/FAIL, Pending/Resolved/Don't Fix. Synonyms = failure.
3. **No preamble** — Skill skips to status field. Text before headers is never read.
4. **Section order matters** — Status line first, issues second.

## Format Violation Consequences

| Violation | Consequence |
|-----------|-------------|
| Missing status field | Workflow stuck, cannot branch |
| Wrong status value | Wrong branch, incorrect execution |
| Wrong section header | Downstream cannot parse, issues lost |
| Extra preamble text | Context ignored, never read |
| Wrong issue ID prefix | ID tracking breaks, duplicates |

## Common Mistakes

| Mistake | Why | What To Do Instead |
|---------|-----|-------------------|
| Adding summary before header | Wanting to explain | Put context in Description field |
| Using synonym for status | Natural language | Use exact: PASS/FAIL |
| Reorganizing sections | Thinking format flexible | Follow template exactly |
| Writing descriptive section names | Thinking names are suggestions | Section names are exact strings |
| Explaining in return message | Wanting to be helpful | Return is file paths only |

## Rationalization Prevention

| Excuse | Reality |
|--------|---------|
| "My format is clearer" | Clearer for you ≠ parseable for skill |
| "I added helpful context at top" | Skill skips preamble. Never read. |
| "Synonym means same thing" | Skill reads literal string. ≠ match. |
| "Template is too rigid" | Rigidity enables automation. |

## Next Step

After reading this skill, proceed to your agent-specific output format section. Write files in exact format specified there.
