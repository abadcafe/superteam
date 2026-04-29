# Superteam

Superteam 是 Superpowers 的改写和扩展，提供轻量级的 AI 驱动开发工作流

## 设计理念

我希望在我跟claude沟通完spec之后，他就不要再打扰我了，自己慢慢跑我睡一觉起来再收货。时间长一点也没关系，不要打扰我睡觉就行。如果不满意，我就删掉让他重新跑，反正电费不值钱，coding plan又是包月的。这样，我就可以没事时就预先搞出一堆东西，在合适的时候拿出来，轻松又愉快🐶

但是superpowers的理念与这冲突很大，他一旦碰上问题就马上跳出来等人类干预，并且他大量使用上下文，长时间跑下来的结果乱七八糟。所以superteam的核心思想就是 Token 换质量：
1. 主agent就是个状态机，不要思考，也不传递除文件路径之外的多余上下文，机械化地执行流程就OK，这样他就能长时间运行。
2. 各个子agent的每次调用都是全新的上下文，从文件系统读取信息来完成该做的事情。
3. 所有 reviewer 都不限制 review 次数。由于通过文件系统沟通并将 issues 持久化记录，迭代必然收敛，无需人为设置上限。
4. 通过黑盒测试来验证最终产出是否符合spec要求
5. 只要模型达到一定水准（如 GLM-5、Qwen 3.6 Plus），就能产出质量相近的代码，主要差别在于 token 消耗和耗时。

目前我自己验证，这套东西在后端开发和 CLI 程序开发中表现良好，其他场景我验证的比较少。欢迎大家提issue和pr。

推荐用高端模型（如 Opus 4.6）生成 plan，再用经济型模型执行实现——这是性价比最优的方案，因为 plan 是流程的关键。

叠个甲：大模型这个东西是概率性的，因此无法保证他万无一失。只要在各种场景下的任务成功率比以往有提升，就是好方法（逃

另外，这套流程是依赖superpowers的，主要是它的tdd skill实在太好了。但是superpowers的其他skill实际上会干扰整体流程执行，因此要加上这些设置：
```
~/.local/bin/claude \
  --disallowedTools \
    "Skill(superpowers:using-superpowers),"\
    "Skill(superpowers:writing-plans),"\
    "Skill(superpowers:executing-plans),"\
    "Skill(superpowers:subagent-driven-development),"\
    "Skill(superpowers:dispatching-parallel-agents),"\
    "Skill(superpowers:systematic-debugging),"\
    "Skill(superpowers:requesting-code-review),"\
    "Skill(superpowers:receiving-code-review),"\
    "Skill(superpowers:verification-before-completion),"\
    "Skill(superpowers:finishing-a-development-branch),"\
    "Skill(superpowers:using-git-worktrees),"\
    "Skill(superpowers:writing-skills),"\
    "Skill(superpowers:brainstorm),"\
    "Skill(superpowers:write-plan),"\
    "Skill(superpowers:execute-plan)" \
  --dangerously-skip-permissions
```

### 后续改进方向

1. planning阶段会耗费较多时间和较多token，因为实际上在plan阶段很多代码都已经写出来了。目前把superpowers的单plan文件模式改成了拆分到多个task目录里的task.md模式，有所改善。
2. executing阶段我特意放弃了superpowers的任务并行开发模式，主要是考虑到无人值守模式下耗时长是可以接受的，反正我不跟他交互。正确性对我来说比耗时更重要, 毕竟就算它并行开发也要3个小时以上, 我不打算半夜起来验证它。
3. 前端等其他场景验证较少。
4. 各种周边系统例如git, ci/cd等, 都没有对接。

## 主要功能

### 最小化上下文实现长时间运行

采用状态机架构设计：
- orchestrator 作为状态机，严格按照流程图执行，不做任何判断或解释
- 所有 subagent 通过文件系统沟通（写入/读取 working/ 目录下的约定文件）
- 每个 subagent 每次调用都是全新 context，不继承任何历史对话

注意：由于大模型是有温度的，所以有时候他仍然会不遵循状态机，此时你重启claude，也许他就会变成
一个守规矩的agent。大部分情况下还是守规矩的。如果你的供应商支持设置温度，也许你可以设置温度
为0。

### 无人值守模式

整个 planning 和 executing 过程完全自动化运行，用户只需在开始前确认 spec/plan 概要，之后无需介入。系统自动处理迭代修复、审核反馈等所有中间环节，直至生成最终的 commit message 和 task summary。

如果你ctrl -c了，他其实也可以继续接着之前的任务跑，你告诉他从几号任务开始执行就好了。

### 黑盒测试能力

黑盒测试主要是为了验证 spec 与实现的一致性：
- 测试用例不参考任何内部实现，只通过外部接口验证功能行为是否符合 spec 定义
- 黑盒测试作为独立任务，与功能实现任务分离，遵循 TDD 方法论

## 工作流

```
1. brainstorming → spec.md
2. planning → plan/
3. executing → changes
```

### Step 1: 需求分析 (Superpowers)

使用你喜欢的 skill（例如`superpowers:brainstorming`, 或者别的方式）定义需求，输出到 `working/spec.md`。

### Step 2: 规划实现 (Superteam)

使用 `planning` skill 根据 spec 创建任务文档。启动方式：

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

根据 `working/spec.md` 创建任务文档到 `working/plan/task-NNN/task.md`。

### executing

执行 `working/plan/` 中的任务，输出 `working/commit-message.md` 和 `working/task-summary.md`。

### black-box-testing（不用手动调用）

黑盒测试规范，用于规划、实现或审核任务。强调：
- 不参考内部实现，只通过外部接口测试
- 优先使用真实工具（curl、nc 等）
- 黑盒测试作为独立任务，遵循 TDD 方法论

### hands-off-issue-handling（不用手动调用）

问题处理规范。当 agent 在执行过程中遇到非工作对象本身的问题（如 spec 模糊、环境异常等），在尝试多种方案后仍无法解决时，需做出假设并记录到文件系统。

三种问题类型：
- `working/spec-issues.md` — 规格文档问题（模糊不清或前后矛盾）
- `working/plan-issues.md` — 任务规划文档问题
- `working/env-issues.md` — 环境问题（编译/运行环境异常）

目的：
- 避免 agent 重复发现已知问题
- 完成后人类仍能了解发生了什么

## Agents

| Agent | 用途 |
|-------|------|
| planner | 创建任务文档 |
| plan-reviewer | 审核任务文档 |
| implementer | 执行任务实现 |
| spec-reviewer | 审核实现是否符合 spec |
| code-reviewer | 审核代码质量 |

## 目录结构

```
.
├── agents/           # 内部 agent 定义
│   ├── planner.md
│   ├── plan-reviewer.md
│   ├── implementer.md
│   ├── spec-reviewer.md
│   └── code-reviewer.md
├── skills/
│   ├── planning/     # /superteam:planning
│   ├── executing/    # /superteam:executing
│   ├── black-box-testing/  # 黑盒测试规范
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

4. plan 写入 `working/plan/`，确认后启动 executing：
   ```
   /superteam:executing
   ```

5. 完成，生成 commit message 和 task summary

## 文件约定

| 文件 | 描述 |
|------|------|
| `working/spec.md` | 需求规格 |
| `working/plan/task-NNN/task.md` | 任务文档 |
| `working/plan-review-results.md` | 计划审核结果 |
| `working/plan/task-NNN/changes.md` | 任务变更 |
| `working/plan/task-NNN/test-results.md` | 测试结果 |
| `working/plan/task-NNN/implement-review-results.md` | 实现审核结果 |
| `working/commit-message.md` | Git commit 消息 |
| `working/task-summary.md` | 任务总结 |
| `working/spec-issues.md` | 规格问题 |
| `working/plan-issues.md` | 计划问题 |
| `working/env-issues.md` | 环境问题 |
