# Git Worktree 集成设计

## 目标

将 superpowers 的 git worktree 功能集成到 superteam，使整个 planning/executing 流程在隔离的 worktree 中运行，最终通过 `finishing-a-development-branch` 收尾。

## 整体流程变更

**之前：**
```
brainstorming → spec.md → /superteam:planning → plan/ → /superteam:executing → commit-message + summary
```

**之后：**
```
brainstorming → spec in docs/superpowers/specs/
  → [using-git-worktrees 自动创建 worktree]
  → /superteam:planning (in worktree)
  → /superteam:executing (in worktree, implementer 每个 task commit)
  → /superpowers:finishing-a-development-branch (合并/PR/清理)
```

## 依赖的 superpowers skill

直接复用，不自建：
- `superpowers:using-git-worktrees` — brainstorming 结束后自动触发，创建 worktree
- `superpowers:finishing-a-development-branch` — executing 结束后调用，处理合并/PR/清理

从 disallowedTools 中移除这两个 skill。

## 改动清单

### 1. planning skill description

**文件：** `skills/planning/SKILL.md`

**改动：** description 从
```
Use when you have a completed spec to create an implementation plan.
```
改为
```
Use after brainstorming when you have a completed spec to create an implementation plan - replaces writing-plans
```

**目的：** 让 brainstorming 结束后自然触发 `superteam:planning` 而非 `superpowers:writing-plans`。

### 2. planning skill 流程：自动发现 spec 并传递路径

**文件：** `skills/planning/SKILL.md`

**改动：** 在状态机流程开头加一步——扫描 `docs/superpowers/specs/` 找最新的 spec 文件，然后在 Agent Prompt Format 中把 `Spec path:` 从写死的 `working/spec.md` 改为动态发现的路径。

**原因：** brainstorming（superpowers skill）把 spec 写到 `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md`，不需要复制到 working/，直接通过 Agent Prompt Format 传路径给 planner 和 plan-reviewer 即可（只有这两个 agent 需要读 spec）。

**具体步骤：**
```bash
# 找最新的 spec 文件
latest_spec=$(ls -t docs/superpowers/specs/*.md 2>/dev/null | head -1)
```

Agent Prompt Format 改为：
```
- Spec path: [dynamically discovered spec file path]
- Plan directory: working/plan/
- Review results path: working/plan-review-results.md
```

File Paths 中删除 `working/spec.md - Spec file` 行。

### 3. implementer agent：每个 task commit + 去掉 changes.md

**文件：** `agents/implementer.md`

**改动：**
- Process Flow Step 3 (Implement) 末尾加 commit 步骤：task 完成并验证后 commit
- commit message 遵循 Conventional Commits 模板（从 executing skill 原有的 `commit-message.md` 输出格式迁移过来）：
  ```
  <type>(<scope>): <subject>

  <body: what changed and why, wrapped at 72 chars>
  ```
  Type: `feat`, `fix`, `refactor`, `perf`, `test`, `docs`, `chore`
  Scope/Subject/Body 从 task.md 的 Project Overview Goal 和 Task Objective 提取
- Response Format 中删除 `working/plan/task-NNN/changes.md`
- 删除整个 `### File: working/plan/task-NNN/changes.md` 输出格式定义
- Step 5 (Write reports) 中删除 `update changes.md` 指令

### 4. code-reviewer agent：改为 git diff review

**文件：** `agents/code-reviewer.md`

**改动：**
- Step 2 (Read Context) 中 `Read changes.md to extract ALL files modified` 改为 `Use git diff (BASE_SHA..HEAD_SHA) to identify all modified files`
- 状态机在 dispatch code-reviewer 时，通过 Agent Prompt Format 传递 BASE_SHA 和 HEAD_SHA

### 5. spec-reviewer agent：不引用 changes.md

**文件：** `agents/spec-reviewer.md`

**改动：**
- Iron Law 中 `changes.md says "implemented"?` 改为 `implementer claims "implemented"?`
- Step 2 (Read Context) 中删除 `Read changes.md to extract ALL files modified` 行，改为 `Read task.md Files section to identify all files to verify`（spec reviewer 需要读完整文件验证是否实现了 spec，不需要 diff，task.md 里已经列了文件路径）

### 6. executing skill：收尾 + 删除废弃输出

**文件：** `skills/executing/SKILL.md`

**改动：**
- File Paths 中删除 `working/plan/task-NNN/changes.md` 行
- 删除整个 `### File: working/commit-message.md` 输出格式定义
- `### File: working/task-summary.md` 中 `[copy from changes.md Files section]` 改为从 git diff 获取文件列表
- "After all tasks finished" 部分删除 `read all working/plan/task-NNN/changes.md` 和 `write working/commit-message.md`
- "After all tasks finished" 末尾加一步：调用 `superpowers:finishing-a-development-branch`
- Agent Prompt Format 中为 code-reviewer 传递 BASE_SHA 和 HEAD_SHA
- 状态机在 dispatch implementer 前记录当前 HEAD SHA 作为 BASE_SHA，implementer 完成后记录新 HEAD SHA 作为 HEAD_SHA，传给 code-reviewer

### 7. hands-off-issue-handling：统一命名

**文件：** `skills/hands-off-issue-handling/SKILL.md`

**改动：** `working/task-issues.md` 保持不变，将 executing skill 和 README 中的 `working/plan-issues.md` 统一改为 `working/task-issues.md`

### 8. README 更新

**文件：** `README.md`

**改动：**
- disallowedTools 列表中移除 `Skill(superpowers:using-git-worktrees)` 和 `Skill(superpowers:finishing-a-development-branch)`
- 文件约定表中删除 `working/plan/task-NNN/changes.md` 和 `working/commit-message.md` 行
- `working/plan-issues.md` → `working/task-issues.md`
- 工作流说明中反映 worktree 自动创建/清理

## 不改动的部分

- **路径：** `working/plan/`、`working/spec-issues.md`、`working/env-issues.md` 等路径全部保持原样。`working/spec.md` 不再使用，spec 路径通过 Agent Prompt Format 动态传递
- **using-git-worktrees skill：** 直接复用 superpowers 的
- **finishing-a-development-branch skill：** 直接复用 superpowers 的
- **planner agent：** 不需要 commit（plan 是过程文件，留在 working/ 里）
- **plan-reviewer agent：** 不需要改动
- **black-box-testing skill：** 无路径引用，不需要改动

## 文件变更总览

| 文件 | 改动类型 |
|------|----------|
| skills/planning/SKILL.md | description + 流程加 spec 发现步骤 |
| skills/executing/SKILL.md | 删 changes.md/commit-message.md + 加 finishing + 传 SHA |
| skills/hands-off-issue-handling/SKILL.md | 统一命名确认 |
| agents/implementer.md | 加 commit + 删 changes.md |
| agents/spec-reviewer.md | 删 changes.md 引用 |
| agents/code-reviewer.md | 改 git diff review |
| README.md | disallowedTools + 工作流 + 文件约定表 |
