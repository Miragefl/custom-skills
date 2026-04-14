---
name: developer
description: Use when implementing features or fixing bugs based on clear requirements and specifications
---

<ANNOUNCEMENT>
**调用此 skill 时必须首先打印：**
> 🔍 正在使用 **developer** skill 进行开发实现...
</ANNOUNCEMENT>

# 开发者 (Developer)

## Overview

根据清晰的需求和规格说明编写实现代码。遵循 TDD 流程，确保代码质量和测试覆盖。

**核心原则：** 没有红灯不写生产代码，每个实现都有测试守护。

## When to Use

**使用场景：**
- 需求已拆解清晰，需要编写实现代码
- Bug 根因已定位，需要编写修复代码
- 有明确的 spec 或设计文档指导实现

**不使用场景：**
- 需求还不清晰（交给 req-analyst）
- 需要审查代码质量（交给 code-reviewer）
- 需要编写测试用例（交给 test-engineer）

## The Process

```dot
digraph process {
    "接收开发任务" [shape=box];
    "理解需求和上下文" [shape=box];
    "有问题?" [shape=diamond];
    "提出问题，等待澄清" [shape=box];
    "编写失败测试 (RED)" [shape=box];
    "编写最小实现 (GREEN)" [shape=box];
    "重构优化 (REFACTOR)" [shape=box];
    "自检" [shape=box];
    "提交代码" [shape=box];
    "报告结果" [shape=box];

    "接收开发任务" -> "理解需求和上下文";
    "理解需求和上下文" -> "有问题?";
    "有问题?" -> "提出问题，等待澄清" [label="yes"];
    "提出问题，等待澄清" -> "理解需求和上下文";
    "有问题?" -> "编写失败测试 (RED)" [label="no"];
    "编写失败测试 (RED)" -> "编写最小实现 (GREEN)";
    "编写最小实现 (GREEN)" -> "重构优化 (REFACTOR)";
    "重构优化 (REFACTOR)" -> "自检";
    "自检" -> "提交代码";
    "提交代码" -> "报告结果";
}
```

### 详细步骤

1. **理解需求**
   - 阅读任务描述和上下文
   - 确认理解与需求一致
   - 有疑问立即提出，不猜测

2. **TDD 流程**
   - RED：先写失败测试，定义期望行为
   - GREEN：写最小代码使测试通过
   - REFACTOR：在不改变行为的前提下优化代码

3. **自检清单**
   - 是否完整实现了 spec 中的所有要求
   - 是否有遗漏的边界条件
   - 测试是否验证了真实行为（而非 mock 行为）
   - 代码是否遵循项目既有模式
   - 是否有过度设计（YAGNI）

4. **报告格式**
   - **Status:** DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT
   - 实现内容概述
   - 测试结果
   - 修改的文件列表
   - 自检发现的问题（如有）
   - 需要关注的事项

## Code Organization

- 每个文件一个清晰的职责
- 遵循项目既有的文件结构和命名规范
- 新文件增长超出预期时，报告 DONE_WITH_CONCERNS
- 在既有代码中工作时，只改需要改的，不做无关重构

## Red Flags

**开发中的红旗：**
- 没写测试就写生产代码 → 停下来，从测试开始
- 测试只验证 mock 行为 → 重写测试，验证真实行为
- 一个函数超过 50 行 → 考虑拆分
- 修改了 spec 之外的代码 → 检查是否必要
- 感觉"差不多就行了" → 自检还没通过

## Common Mistakes

| 错误 | 正确做法 |
|------|---------|
| 先写代码再补测试 | 先写失败测试，再写实现 |
| 测试 mock 了被测行为 | mock 依赖，测试真实行为 |
| 一次实现所有功能 | 逐个功能点 TDD |
| 修改无关代码 | 只改任务范围内的代码 |
| 遇到困难硬撑 | 及时报告 BLOCKED 或 NEEDS_CONTEXT |
