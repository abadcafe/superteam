# Superteam

Superteam 是 Superpowers 的改写和扩展，提供轻量级的 AI 驱动开发工作流

## 设计理念

我希望在我跟claude沟通完spec之后，他就不要再打扰我了，自己慢慢跑我睡一觉起来再收货。时间长
一点也没关系，不要打扰我睡觉就行。如果不满意，我就删掉让他重新跑，反正电费不值钱，coding
plan又是包月的。这样，我就可以没事时就预先搞出一堆东西，在合适的时候拿出来，轻松又愉快🐶

但是superpowers有这几个问题：
1. 一旦碰上问题就马上跳出来等人类干预
2. 大量使用上下文，长时间跑下来的结果与预期不符
3. 架构设计方面完全没有约束，模块之间的边界等全部随意来，项目基本上是不可维护的
4. 最重要的一点，它的plan里把所有代码都写完了，既费时间又费token, 可是实际上质量很低

所以superteam的核心思想就是 Token 换质量：
1. 主agent就是个状态机，不要思考，也不传递除文件路径之外的多余上下文，机械化地执行流程就OK，这样他就能长时间运行。
2. 各个子agent的每次调用都是全新的上下文，从文件系统读取信息来完成该做的事情。
3. 所有 reviewer 都不限制 review 次数。由于通过文件系统沟通并将 issues 持久化记录，迭代必然收敛，无需人为设置上限。
4. 测试定义行为——测试是模块行为的唯一权威定义，实现必须通过测试
5. 只要模型达到一定水准（如 GLM-5、xiaomi mimo v2.5），就能产出质量相近的代码，主要差别在于 token 消耗和耗时。

目前我自己验证，这套东西在后端开发和 CLI 程序开发中表现良好，其他场景我验证的比较少。欢迎大家提issue和pr。

推荐用高端模型（如 Opus 4.6）生成 plan，再用经济型模型执行实现——这是性价比最优的方案，因为 plan 是流程的关键。

叠个甲：大模型这个东西是概率性的，因此无法保证他万无一失。只要在各种场景下的任务成功率比以往有提升，就是好方法（逃

### 后续改进方向

1. 前端等其他场景验证较少。

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

整个 planning 和 executing 过程完全自动化运行，用户只需在开始前确认 spec/plan 概要，之后
无需介入。系统自动处理迭代修复、审核反馈等所有中间环节，直至生成最终的 task summary。

如果你ctrl -c了，他其实也可以继续接着之前的任务跑，你告诉他从几号任务开始执行就好了。

### 模块设计与 TDD

模块设计确保代码结构合理，TDD 确保行为定义清晰：
- 模块遵循单一职责和接口最小化原则
- 测试定义模块行为，实现必须通过测试
- 测试模块与被测模块分离，只通过公开接口交互

## 工作流

1. 用claude -w进入一个worktree
2. 写个 working/spec.md
3. /planning (可以指定spec的路径，默认是working/spec.md) → working/plan/（在 worktree 中）
4. /executing → commit per task + task-summary（在 worktree 中）

## Skills

### planning

- 根据 spec 创建任务文档到 `working/plan/task-NNN/task.md`
- 作为状态机驱动 planner 和 plan-reviewer 迭代，直至所有 issues 解决

### executing

- 串行执行 `working/plan/` 中的任务，每个 task 完成后 commit
- 全部完成后输出 `working/task-summary.md`
- 作为状态机驱动 implementer、task-reviewer 迭代，直至所有 issues 解决

### module-design（不用手动调用）

模块设计规范，用于规划、实现或审核任务。强调：
- 单一职责：一个模块只有一个变更理由
- 接口最小化：公开接口尽量少
- 黑盒测试：测试模块只允许调用被测模块的公开接口
- 测试模块与被测模块必须分离（`xxx.rs` + `xxx_tests.rs`）
- 集成测试放在 tests 目录，覆盖所有端到端用户场景，禁止 mock

### hands-off-issue-handling（不用手动调用）

问题处理规范。当 agent 在执行过程中遇到非工作对象本身的问题（如 spec 模糊、环境异常等），在
尝试多种方案后仍无法解决时，需做出假设并记录到文件系统。

三种问题类型：
- `working/spec-issues.md` — 规格文档问题（模糊不清或前后矛盾）
- `working/task-issues.md` — 任务规划文档问题
- `working/env-issues.md` — 环境问题（编译/运行环境异常）

目的：
- 避免 agent 重复发现已知问题
- 完成后人类仍能了解发生了什么

## Agents

| Agent | 用途 |
|-------|------|
| planner | 根据 spec 创建任务文档，遵循 TDD 和模块设计规范 |
| plan-reviewer | 审核任务文档的完整性、spec 对齐和可构建性 |
| implementer | 按 TDD 流程执行单个任务：写测试 → 实现 → commit |
| task-reviewer | 审核单个任务实现的 task 符合度和代码质量 |

## 目录结构

```
.
├── agents/           # 内部 agent 定义
│   ├── planner.md
│   ├── plan-reviewer.md
│   ├── implementer.md
│   └── task-reviewer.md
├── skills/
│   ├── planning/     # /superteam:planning
│   ├── executing/    # /superteam:executing
│   ├── module-design/      # 模块设计与黑盒测试规范
│   └── hands-off-issue-handling/  # 内部问题处理
└── README.md
```

## 文件约定

| 文件 | 描述 |
|------|------|
| `docs/superpowers/specs/*.md` | 需求规格（brainstorming 输出） |
| `working/plan/task-NNN/task.md` | 任务文档 |
| `working/plan-review-results.md` | 计划审核结果 |
| `working/plan/task-NNN/test-results.md` | 测试结果 |
| `working/plan/task-NNN/implement-review-results.md` | 实现审核结果 |
| `working/plan/task-NNN/test-case-changes.md` | 测试用例变更记录 |
| `working/task-summary.md` | 任务总结 |
| `working/spec-issues.md` | 规格问题 |
| `working/task-issues.md` | 任务文档问题 |
| `working/env-issues.md` | 环境问题 |
