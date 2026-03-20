# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Claude Code plugin called "superteam" - a multi-agent software development system. It defines specialized agents that work together through a structured workflow: requirements gathering → architecture design → planning → implementation → testing.

## User-Invocable Skills

The primary interface to this plugin is through three skills:

- `/I-have-ideas` - Start as product-manager to gather requirements and create `working/requirements.md`
- `/design` - Run architecture design workflow (architect → review → plan → review)
- `/execute` - Execute the implementation plan with the execution team

## Architecture

### Communication Model

**Workers communicate only through the file system** - they do not call each other directly. Managers orchestrate workers by reading their output documents and invoking them via the Agent tool.

### Teams

**Product Manager** (standalone):
- Gathers requirements, writes `working/requirements.md`

**Architecture Team** (managed by architecture-manager via `/design`):
- `architect` - Designs system architecture, writes `working/design.md`
- `architecture-reviewer` - Reviews architecture document
- `planner` - Creates implementation plan, writes `working/plan.md`
- `plan-reviewer` - Reviews implementation plan

**Execution Team** (managed by execution-manager via `/execute`):
- `coder` - Implements code with unit tests (100% coverage required)
- `code-reviewer` - Reviews code and unit tests
- `tester` - Writes and runs integration tests using Python/pytest
- `test-reviewer` - Reviews integration tests
- `analyst` - Analyzes failures and identifies root cause tasks

### Task Types

There are exactly 4 task types in plans:
- **代码编写任务**: Coder implements a feature with unit tests
- **代码审查任务**: Code Reviewer reviews the implementation
- **集成测试任务**: Tester writes and runs integration tests for a feature
- **集测审查任务**: Test Reviewer reviews integration tests

### Agent Definitions

Agents are defined in `agents/*.md` with frontmatter specifying:
- `model`: Model to use (e.g., `glm-5`)
- `tools`: Available tools
- `skills`: Inherited skills (typically `supreme-constraints`)
- `permissionMode`: Usually `bypassPermissions`

### Skills Structure

```
skills/
├── supreme-constraints/SKILL.md  # Universal constraints all agents must follow
│   ├── format/                   # Document format specifications
│   └── quality/                  # Quality standards (code, test, design)
├── design/SKILL.md               # Architecture workflow manager
├── execute/SKILL.md              # Execution workflow manager
└── I-have-ideas/SKILL.md         # Requirements gathering
```

## Critical Constraints (from supreme-constraints)

### File System
- **Write operations** only affect files in current working directory
- **Temp files** must use `tmp/` in current directory, never `/tmp`
- **Read operations** allowed from: current directory, home directory, `/usr`

### Document Paths and Ownership

All documents live in `working/`. **Strict ownership rules apply** - each document can only be written by specific roles:

| Document | Writer |
|----------|--------|
| `requirements.md` | product-manager only |
| `design.md` | architect only |
| `design-review-results.md` | architecture-reviewer only |
| `plan.md` | planner only |
| `plan-review-results.md` | plan-reviewer only |
| `commit-message.md`, `task-summary.md` | execution-manager only |
| `requirement-issues.md` | architecture-manager or execution-manager |
| `design-issues.md` | execution-manager only |
| `working/artifacts/task-{NNN}/*` | Workers (coder, tester, reviewers, analyst) |

### Code Quality
- All Python code must have complete type annotations
- Unit test coverage must be 100%
- Follow existing code style or standard library conventions

### Integration Testing
- Black-box tests only (no internal code calls)
- Use Python with pytest
- Tests located in `tests/integration/`
- Use real tools (curl, nc, socat) to test external interfaces
- Test code must be written as proper pytest files, not inline scripts

### Document Formats
All documents have strict formats defined in `skills/supreme-constraints/format/`. Agents must:
- Follow formats exactly - no additions or modifications
- Never use diagrams or tables - use markdown lists instead
- Write in UTF-8 encoding

## Workflow Summary

1. `/I-have-ideas` → Create `working/requirements.md`
2. `/design` → Create `working/design.md` and `working/plan.md` (with review cycles)
3. `/execute` → Implement tasks sequentially (no parallelism):
   - Coder writes code + unit tests (100% coverage)
   - Code reviewer checks code
   - Tester writes/runs integration tests
   - Test reviewer checks tests
   - On failure: Analyst finds root cause, reset affected tasks to `⏳ 待开始`
