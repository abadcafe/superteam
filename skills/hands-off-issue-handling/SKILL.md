---
name: hands-off-issue-handling
description: Use when issues require escalation
user-invocation: false
---

# Hands Off Issue Handling

## 概览

在写代码, 做测试, 做任务规划文档(plan.md), 以及各种review工作的时候, 会发现各种问题
(issue). 有些问题是写的或被审查的代码或方案本身的bug, 有些问题则是运行环境或前置工作引起的,
后者就需要agent自主决策如何处理. 而agent在做出这些决策后, 也必须将这些问题记录下来,一方面避
免不同的agent重复发现已知问题, 另一方面在agent自主完成所有工作完成之后, 人类仍然能知道发生
了什么事情.

**只按条记录问题, 禁止记录任何其他信息:**
  - "no issue found" - 这类信息没有任何意义, **绝对禁止**
  - "Notes" - 这类信息与真正的问题无关, **绝对禁止**

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
  - 问题有规避方案 - 工作对象本身的问题

### 规格文档问题列表

在做任务规划文档(plan.md)的时候, 需要遵循规格文档(spec.md). 如果规格文档中有模糊不清或前后
矛盾的地方, 那么agent就要在做出合理假设后记录这些问题.

记录格式:

```markdown
# Spec Issues

## SI-001: [title]
- **Description**: [ambiguity/gap in spec]
- **Assumption**: [what we assume to proceed]
```

### 任务规划文档问题列表

在编写代码和测试的时候, 需要遵循任务规划文档(plan.md). 如果任务规划文档中有模糊不清或前后
矛盾的地方, 那么agent就要在做出合理假设后记录这些问题.

记录格式:

```markdown
# Plan Issues

## PI-001: [title]
- **Description**: [issue with plan]
- **Assumption**: [what we assume to proceed]
```

### 环境问题列表

在编译代码和测试的时候, 可能会因为当前运行环境的问题而失败. agent必须在尝试3种以上解决方案之
后, 才可以将这种问题归类为环境问题, 记录下来并继续其他工作.

记录格式:

```markdown
# Environment Issues

## EI-001: [title]
- **Description**: [what is unavailable/mismatched]
- **Assumption**: [what we assume or none]
```
