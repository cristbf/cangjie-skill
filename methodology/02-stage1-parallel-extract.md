# 阶段 1 — 6 个 sub-agent 并行提取

## 目标

不用单一视角读一遍,而是**同时从 6 个不同角度扫描全书**,最大化候选单元覆盖率。

## 为什么要并行

- **覆盖**: 单一视角会漏。框架提取器找不到的"反例",反例提取器会找到。
- **速度**: Claude Code 的 Agent 工具支持并行,不用白不用。
- **独立性**: 每个 extractor 独立判断,避免互相污染 — 三重验证才能真正起作用 (V1 跨域要求"独立出现")

## 6 个 sub-agent

每个 sub-agent 接收:
- `BOOK_OVERVIEW.md` (阶段 0 产出, 提供全局上下文)
- 书本文本 (或文本路径)
- 对应的 extractor prompt (`extractors/<type>-extractor.md`)

并在一次调用中通过 Agent 工具 **同时 spawn 6 个**,不是串行。

| # | extractor | 查找对象 | 产出文件 |
|---|---|---|---|
| 1 | framework-extractor | 思维模型 / 决策框架 / 推理方法 | `candidates/frameworks.md` |
| 2 | principle-extractor | 原则 / 清单 / 规则 / 断言 | `candidates/principles.md` |
| 3 | case-extractor | 作者在书中亲自使用的实例 | `candidates/cases.md` |
| 4 | counter-example-extractor | 作者警告的失败 / 反例 / 陷阱 | `candidates/counter-examples.md` |
| 5 | glossary-extractor | 关键概念词典 | `candidates/glossary.md` |
| 6 | tension-extractor | 内在矛盾 / 未解问题 / 未深入的反对观点 | `candidates/tensions.md` |

### 条件提取器（根据书籍类型决定是否启用）

| 书籍类型 | 替换/新增 | 使用的提取器 | 产出文件 |
|---------|----------|-------------|---------|
| B 类（叙事型） | 替换 #1 | `pattern-extractor`（替代 framework-extractor） | `candidates/patterns.md` |
| C 类（理论型） | 额外新增 | `argument-chain-extractor`（额外运行） | `candidates/argument-chains.md` |

条件提取器的详细说明见 SKILL.md 的阶段 -1 书籍类型判断部分。

## 每个候选单元的最小字段

无论是哪个 extractor,产出的每条候选单元必须包含:

```yaml
id: f01                           # 类型缩写 + 序号
title: 逆向思维                    # 简短标题
type: framework                   # framework / principle / case / counter-example / term / tension / pattern / argument_chain
source_chapter: 第三讲             # 书中位置
source_quote: |                   # 原文引用 ≤150 字
  "反过来想,总是反过来想..."
summary: |                        # 用自己的话,5-10 行
  ...
tags: [decision, mental-model]    # 便于后续链接
```

## 输出前的自检

每个 extractor 在提交候选之前自问:
1. 这个单元**在书中**有明确根据吗? (不是我脑补)
2. 它属于我这个 extractor 的职责范围吗? (不要越界)
3. 它是不是已经在别处被别的 extractor 提取过了? (重复不是问题,阶段 1.5 会合并)

## 不在本阶段做的事

- **不做筛选** — 宁错杀,留给阶段 1.5 三重验证
- **不写 skill** — 只出候选,不出 SKILL.md
- **不做跨单元链接** — 留给阶段 3
