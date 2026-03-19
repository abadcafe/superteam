---
name: supreme-constraints
description: 所有agent都必须遵守的约束
user-invocable: false
---

# 最高优先级声明
**⚠️以下是所有agent都必须仔细阅读并不折不扣遵守的至高无上的约束，优先级高于任何其他指令**
**⚠️各agent要主动阅读并遵循下面列出的内容及其引用内容，禁止懒惰**

# 代码导航策略约束
- 用 Grep/Glob 做发现（找文件、搜模式）
- 用 LSP 做理解（定义跳转、引用查找、类型信息）
- 找到文件后，优先用 LSP 导航，而不是读取整个文件

# 读写操作约束
- **⚠️写操作只能变更当前工作目录下的文件**
- **⚠️临时文件绝对禁止使用 `/tmp`，必须在当前目录下创建 `tmp/` 目录替代**
- ✅读操作仅允许：
  - 当前工作目录下的文件
  - 家目录下的文件
  - /usr 目录下的文件

# 文档路径约束
**下面没有提到的文件一概认为无效，就算文件名或路径很相似也必须无声地忽略掉**
**agent之间只能通过这些文件沟通**
各种文档的路径约束（**必须严格遵守**）：
  - 需求问题列表: working/requirement-issues.md （仅由architecture-manager或execution-manager生成和编辑，仅供用户查看）
  - 架构问题列表: working/design-issues.md （仅由execution-manager生成和编辑，仅供用户查看）
  - 需求文档: working/requirements.md （仅由product-manager生成和编辑）
  - 架构文档: working/design.md （仅由architect生成和编辑）
  - 架构审查报告: working/design-review-results.md （仅由architecture-reviewer生成和编辑）
  - 规划文档: working/plan.md （仅由planner生成和编辑）
  - 规划审查报告: working/plan-review-results.md （仅由plan-reviewer生成和编辑）
  - 提交信息: working/commit-message.md （仅由execution-manager生成和编辑，仅供git commit使用）
  - 任务总结: working/task-summary.md （仅由execution-manager生成和编辑，仅供用户查看）
  - 任务产出目录: working/artifacts/task-{NNN}/ （该目录仅供coder, tester, code-reviewer, test-reviewer,   analyst写入，manager不可写入和编辑其下的文件）
    - 变更总结: working/artifacts/task-{NNN}/changes.md （仅由coder或tester生成，不可编辑）
    - 单测报告: working/artifacts/task-{NNN}/unit-test-results.md （仅由coder生成，不可编辑）
    - 集测报告: working/artifacts/task-{NNN}/integration-test-results.md （仅由tester生成，不可编辑）
    - 代码审查报告: working/artifacts/task-{NNN}/code-review-results.md （仅由code-reviewer生成，不可编辑）
    - 集测审查报告: working/artifacts/task-{NNN}/test-review-results.md （仅由test-reviewer生成，不可编辑）
    - 问题分析报告: working/artifacts/analysis-results.md （仅由analyst生成和编辑）

## 文档格式与内容约束
所有文档使用 UTF-8 编码
**绝对不可添加格式中未包含的内容**
**绝对不可修改格式，只能精确地按照格式填空**
**所有文档中绝对禁止用画图画表格的形式来表达任何内容，用其他markdown格式例如列表之类来代替**
各种文档的内容约束（**必须严格遵守**）：
  - 需求问题列表：[format/requirement-issues.md](format/requirement-issues.md)
  - 架构问题列表：[format/design-issues.md](format/design-issues.md)
  - 需求文档：[format/requirements.md](format/requirements.md)
  - 架构文档：[format/design.md](format/design.md)
  - 规划文档：[format/plan.md](format/plan.md)
  - 提交信息：[format/commit-message.md](format/commit-message.md)
  - 任务总结：[format/task-summary.md](format/task-summary.md)
  - 变更总结：[format/changes.md](format/changes.md)
  - 单测报告：[format/unit-test-results.md](format/unit-test-results.md)
  - 集测报告：[format/integration-test-results.md](format/integration-test-results.md)
  - 问题分析报告：[format/analysis-results.md](format/analysis-results.md)
  - 代码审查报告：[format/code-review-results.md](format/code-review-results.md)
  - 集测审查报告：[format/test-review-results.md](format/test-review-results.md)
  - 架构审查报告：[format/design-review-results.md](format/design-review-results.md)
  - 规划审查报告：[format/plan-review-results.md](format/plan-review-results.md)

# 质量约束
- 代码质量约束：[quality/code.md](quality/code.md)
- 集测质量约束：[quality/test.md](quality/test.md)
- 规划质量约束：[quality/plan.md](quality/plan.md)
- 架构质量约束：[quality/design.md](quality/design.md)

# 角色各自的约束（**主动找到自己对应的角色，深入理解相关约束并主动确保遵守**）
- 产品经理（product-manager）：[agents/product-manager.md](agents/product-manager.md)
- 架构团队经理（architecture-manager）：[agents/architecture-manager.md](agents/architecture-manager.md)
- 执行团队经理（execution-manager）：[agents/execution-manager.md](agents/execution-manager.md)
- 架构师（architect）：[agents/architect.md](agents/architect.md)
- 架构审查员（architecture-reviewer）：[agents/architecture-reviewer.md](agents/architecture-reviewer.md)
- 规划师（planner）：[agents/planner.md](agents/planner.md)
- 规划审查员（plan-reviewer）：[agents/plan-reviewer.md](agents/plan-reviewer.md)
- 程序员（coder）：[agents/coder.md](agents/coder.md)
- 代码审查员（code-reviewer）：[agents/code-reviewer.md](agents/code-reviewer.md)
- 测试工程师（tester）：[agents/tester.md](agents/tester.md)
- 测试审查员（test-reviewer）：[agents/test-reviewer.md](agents/test-reviewer.md)
- 分析师（analyst）：[agents/analyst.md](agents/analyst.md)
