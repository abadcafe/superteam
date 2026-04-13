---
name: black-box-testing
description: Use when planning, implementing or reviewing tasks
user-invocation: false
---

# Black-box testing

做测试的时候, 要注意黑盒测试(black-box tests)和白盒测试(white-box tests)的区别. 黑盒测
试有如下性质:
1. 绝不参考任何内部实现, 只通过外部接口测试, **坚决禁止**调用被测系统内部代码
2. 优先使用真实工具（curl、nc 等）

黑盒测试也被叫做集成测试或者端到端测试, 白盒测试也被叫做单元测试.

## 黑盒测试任务规划(planning)与TDD(Test Driven Development)

做任务规划时, 不允许把同一个功能的白盒测试和功能实现划分成不同的任务, 但必须把黑盒测试划分成
独立任务. 这是为了遵循黑盒测试的原则 - 黑盒测试不关心任何内部实现而只验证功能和场景, 因此它
根本不对应内部的功能模块, 而任务规划则是按功能模块来划分的.

黑盒测试同样遵循TDD方法论(use `superpowers:test-driven-development`), 但它不是在一个
任务内进行TDD循环, 而是在任务间进行大的TDD循环.

**黑盒测试遵循TDD方法论示例:**
```
Task 1: [write/update black-box tests and see them failed]
Task 2: [write/update unit tests and implementions according to TDD]
...
Task N: [run black-box tests, verify if implementions works, do bug fix]
```

- Task 1与Task N必须是2个任务, 否则即违反TDD
- Task 1只能在相关的代码编写任务之前完成, 否则它可能不会失败, 从而违反TDD
- Task N只能在相关的代码编写任务之后完成, 否则无法验证到相关的代码, 从而违反TDD

**注意:**
- 测试用例本身的函数编写过程不用遵循TDD, 但是辅助函数, 工具等仍然要遵循TDD
- 运行测试和验证实现的任务必须也要负责找出导致测试用例**非预期**结果的根因, 然后修复(修复过程也要TDD)并再次验证
  - 毕竟测试用例或代码实现都有可能出bug
- 所有黑盒测试用例加起来必须完整覆盖所有用户场景(正向/负向场景以及各种边界条件)和所有外部接口

## 黑盒测试规范

1. 严格遵循项目现有的黑盒测试语言框架以及目录约定, 若无既定惯例则默认使用python + pytest并遵循pytest的目录约定
2. 所有测试用例必须写成代码，**禁止手工命令或临时脚本形式的测试用例**
3. 所有测试 mock 必须按测试框架的要求写成代码（例如 pytest fixture）
4. 所有测试代码必须规范地保存到测试代码目录合适位置
  - 例如，所有公共 fixture 都在 conftest.py 中定义
5. 测试用例优先选择调用真实工具（如curl、nc等）而不是自己编写类似功能, 因为这样更接近真实场景
  - 设计测试用例前**必须**先使用 `[command] --help` 等手段充分理解真实工具的所有参数和行为, **禁止**基于推测的行为设计测试用例
6. 测试用例**禁止**参考任何功能代码和单测代码, 只能从外部接口的角度来完成测试
7. 所有 Python 代码**必须**有完整的 Type Annotation
8. 确保被测的可执行文件是最新版本且正确运行
9. 规范管理测试产出物，不污染无关目录和环境
10. 测试配置文件应作为测试代码的一部分，不能污染正常配置文件目录
11. 测试用例可独立运行（无用例间依赖）
12. 测试用例可重复运行（无状态残留）
13. 禁止假设测试环境是自己独占, 尽量避免潜在环境冲突.
