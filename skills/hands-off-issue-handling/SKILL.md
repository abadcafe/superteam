---
name: hands-off-issue-handling
description: Use when issues require escalation
user-invocation: false
---

# Hands-Off Issue Handling

在写代码, 做测试, 做任务规划文档(plan.md), 以及各种review工作的时候, 会发现各种问题
(issue). 有些问题是写的或被审查的代码或文档本身的bug, 有些问题则是运行环境或前置工作引起的,
后者就需要agent在竭尽全力执行3种以上不同的解决方案之后, 再自主假设如何处理. 而agent一旦做出
假设, 就必须将问题和假设都记录下来, 一方面避免不同的agent重复发现已知问题, 另一方面在完成所
有工作之后, 人类仍然能知道发生了什么事情.

**只按条记录问题, 禁止记录任何其他信息:**
  - "no issue found" - 这类信息没有任何意义, **绝对禁止**
  - "Notes" - 这类信息与真正的问题无关, **绝对禁止**
  - 不知道要写什么就马上住手!

## 问题记录文件路径

规格文档问题列表: working/spec-issues.md
任务规划文档问题列表: working/plan-issues.md
环境问题列表: working/env-issues.md

## 读取已知问题

**在进行各项工作的过程中, 必须充分考虑所有已知问题的已有假设(Assumption). 因此:**
  - 编写或review任务规划文档之前, 要先读规格文档问题列表.
  - 编写, 运行或review代码和测试用例之前, 要先读任务规划文档问题列表和环境问题列表.

## 记录新问题

- **已经记录的问题禁止重复记录**
- 追加记录新问题的时候要描述清楚问题和所做的假设
- 工作对象(代码, 任务规划文档, 测试用例等)本身的问题不做记录:
  - 问题可以不做任何假设即自主解决 - 工作对象本身的问题
  - 问题有解决方案 - 工作对象本身的问题

### 规格文档问题列表

在编写任务规划文档(plan.md)的时候, 需要遵循规格文档(spec.md), 如果规格文档中有模糊不清或
前后矛盾的地方, 那么agent就必须自己做出合理假设来解决并记录问题.

**注意:**
  - 规划者违反规格文档不是规格文档的问题, 必须马上解决, 禁止记录下来不处理
  - 编写规格文档时发现的规格文档问题必须马上解决, 禁止记录下来不处理

记录格式:

```markdown
# Spec Issues

## SI-001: [title]
- **Description**: [ambiguity/gap in spec]
- **Assumption**: [what we assume to proceed]
```

### 任务规划文档问题列表

在编写代码和测试的时候, 需要遵循任务规划文档(plan.md), 如果任务规划文档中有模糊不清或前后矛
盾的地方, 那么agent就必须自己做出合理假设来解决并记录问题.

**注意:**
  - 实现者违反任务规划文档所引起的问题不是任务规划文档的问题, 必须马上解决, 禁止记录下来不处理
  - 编写任务规划文档时发现的任务规划文档问题, 必须马上解决, 禁止记录下来不处理
  - 已有代码中的bug就算任务规划文档中没有提到也不属于其问题, 必须主动解决bug, 禁止记录下来不处理

记录格式:

```markdown
# Plan Issues

## PI-001: [title]
- **Description**: [issue with plan]
- **Assumption**: [what we assume to proceed]
```

### 环境问题列表

在编译代码和测试的时候, 可能会因为当前运行环境的问题而失败, 必须在探索并执行3种以上不同的解
决方案仍然失败之后, 才可以归类为环境问题记录下来并继续其他工作.

**注意:**
  - 任何代码逻辑问题都绝不属于环境问题, 必须马上解决, 禁止记录下来不处理

记录格式:

```markdown
# Environment Issues

## EI-001: [title]
- **Description**: [what is unavailable/mismatched]
- **Assumption**: [what we assume or none]
```
