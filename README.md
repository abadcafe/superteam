# superteam

Superteam 是 Superpowers 的改写和扩展，提供完整的 AI 驱动开发工作流。

## 工作流

```
1. brainstorming → spec.md
2. planning → plan.md
3. executing → changes
```

### Step 1: 需求分析 (Superpowers)

使用 `superpowers:brainstorming` skill 定义需求，输出到 `working/spec.md`。

### Step 2: 规划实现 (Superteam)

使用 `planning` skill 根据 spec 创建实现计划。启动方式：

```bash
claude --plugin-dir /path/to/superteam
```

然后在对话中使用 skill：

```
/superteam:planning
```

### Step 3: 执行实现 (Superteam)

使用 `executing` skill 执行计划：

```
/superteam:executing
```

## Skills

### planning

根据 `working/spec.md` 创建实现计划 `working/plan.md`。

### executing

执行 `working/plan.md` 中的任务，输出 `working/commit-message.md` 和 `working/task-summary.md`。

## 目录结构

```
.
├── agents/           # 内部 agent 定义
├── skills/
│   ├── planning/     # /superteam:planning
│   ├── executing/   # /superteam:executing
│   └── hands-off-issue-handling/  # 内部问题处理
└── README.md
```

## 使用方法

### 安装

```bash
claude --plugin-dir /path/to/superteam
```

### 开始项目

1. 启动 brainstorming：
   ```
   /superpowers:brainstorming
   ```

2. spec 写入 `working/spec.md`

3. 启动 planning：
   ```
   /superteam:planning
   ```

4. plan 写入 `working/plan.md`，确认后启动 executing：
   ```
   /superteam:executing
   ```

5. 完成，生成 commit message 和 task summary

## 文件约定

| 文件 | 描述 |
|------|------|
| `working/spec.md` | 需求规格 |
| `working/plan.md` | 实现计划 |
| `working/plan-review-results.md` | 计划审核结果 |
| `working/artifacts/task-NNN/changes.md` | 任务变更 |
| `working/artifacts/task-NNN/test-results.md` | 测试结果 |
| `working/artifacts/task-NNN/implement-review-results.md` | 实现审核结果 |
| `working/commit-message.md` | Git commit 消息 |
| `working/task-summary.md` | 任务总结 |
| `working/spec-issues.md` | 规格问题 |
| `working/plan-issues.md` | 计划问题 |
| `working/env-issues.md` | 环境问题 |