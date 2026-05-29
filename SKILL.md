---
name: book2skill
description: Distill a book into a coherent set of executable skills. Use when the user asks to "拆书" / "蒸馏一本书" / "把 XX 书做成 skill" / "turn a book into skills" — i.e. wants a book's frameworks, principles, and methodologies extracted into atomic, reusable Claude skills that an agent can invoke in real-world situations. NOT for simple summarization, book reviews, or role-playing as the author (that is nuwa-skill's job).
---

# book2skill — 把一本书蒸馏成一组可执行 skills 的元 skill

## 使命

把一本书里沉淀的方法论,拆解成一组**原子化、可被 agent 在真实场景下调用**的 skills,让读者真正用起来。

**边界**:
- ✅ 做: 方法论 / 决策框架 / 清单 / 原则 / 概念体系的蒸馏
- ✅ 做: 内在矛盾 / 未解问题 / 作者盲点的识别（写入 skill 的 Boundary）
- ❌ 不做: 书摘 / 读后感 / 作者人设角色扮演 (后者请用 nuwa-skill)
- ❌ 不做: 体验型书籍（文学/散文/诗歌）— 其知识是体验性的,不能提取成可执行方法论

## 核心方法论: RIA-TV++

一个四阶段 + 并行提取 + 三重验证 + darwin 兼容测试的流水线。详见 `methodology/00-overview.md`。

```
阶段 -1: 书籍类型判断         → 框架型/叙事型/理论型/体验型
阶段 0:  Adler 整书理解       → BOOK_OVERVIEW.md
阶段 1:  6 个 agent 并行提取  → 候选方法论单元池
阶段 1.5: 三重验证筛选        → 通过的单元
阶段 2:  RIA++ 构造 skill     → 每个 skill 的 SKILL.md
阶段 3:  Zettelkasten 链接    → INDEX.md
阶段 4:  压力测试 (darwin 兼容) → test-prompts.json + 回炉淘汰
阶段 5:  反馈整合 (可选)       → feedback.json + 迭代改进
```

## 何时调用此 skill

用户说类似:
- "帮我拆《穷查理宝典》"
- "把毛选蒸馏成 skill"
- "distill this book into skills: <path>"
- "我想把这本书的方法论做成可用的 skill"

## 输入要求

在开始前**必须**从用户处确认:
1. **书的文本来源**: PDF / EPUB / TXT 文件路径, 或可访问的纯文本。**不要**在没有文本的情况下"凭记忆"拆书 — 宁可停下来问用户要。
2. **书名 + 作者 + 出版年**: 用于目录命名和审计。
3. **是否首次试点**: 如果用户是第一次用 book2skill,建议先拆 1 本验证流程再批量。

## 输出结构

```
books/<book-slug>/
├── BOOK_OVERVIEW.md           # 阶段 0 产出: 主旨/骨架/术语/批判
├── INDEX.md                   # 阶段 3 产出: skill 总览 + 引用图
├── candidates/                # 阶段 1 产出: 原始候选池 (审计用)
├── rejected/                  # 阶段 1.5 淘汰的单元 + 原因 (审计用)
├── <skill-slug-1>/
│   ├── SKILL.md
│   └── test-prompts.json      # darwin-skill 兼容格式
├── <skill-slug-2>/
│   └── ...
```

## 执行流程 (严格按顺序)

### 阶段 -1 — 书籍类型判断

在开始蒸馏之前,先判断这本书属于哪种类型,因为不同类型需要不同的处理策略。

根据书名、目录、前言,判断属于以下哪种:

| 类型 | 典型书籍 | 流水线调整 |
|------|---------|-----------|
| **A. 框架型** | 商业/管理/投资/决策类 | 标准 RIA-TV++ 流水线,无调整 |
| **B. 叙事型** | 历史/传记/纪实类 | 见下方"B 类调整" |
| **C. 理论型** | 哲学/科学/学术类 | 见下方"C 类调整" |
| **D. 体验型** | 文学/散文/诗歌类 | 不适用 book2skill,建议用 nuwa-skill |

判断结果写入 `BOOK_OVERVIEW.md` 头部:

```yaml
book_type: A  # A/B/C/D
type_rationale: "本书以框架和原则为主,每个章节独立提出一个可操作的方法论"
pipeline_adjustments: none  # 或列出具体调整项
```

#### B 类调整（叙事型）

- **framework-extractor → pattern-extractor**: 不只找作者命名的框架,还找**反复出现的行为模式**（同一模式出现 3 次以上）
- **V1 调整**: 跨域验证从"2 个独立段落"改为"2 个独立事件/时期/案例"
- **新增隐含方法论提取信号**:
  - 人物在关键时刻的决策过程（"他为什么在这个时候选择 X 而不是 Y"）
  - 事后被证明正确的反直觉选择
  - 同一模式在不同情境下的重复出现

#### C 类调整（理论型）

- **引用限制放宽**: R 段总量上限从 500 字提升到 800 字,允许保留完整推导链
- **新增 argument-chain-extractor**: 提取从前提到结论的推理路径
- **E 段调整**: 理论型 skill 的 E 段不写"操作步骤",写"论证复现步骤"（"理解这个论证需要先理解 X,然后..."）
- **V2 调整**: 预测力测试改为"能否用这个推理路径分析一个新的论题"

#### D 类处理（体验型）

不适用 book2skill。告诉用户:
> 这是一本体验型书籍（文学/散文/诗歌）,其知识是体验性的,不能提取成可执行方法论。
> 建议使用 nuwa-skill 蒸馏作者的思维风格和表达 DNA。
> 如果你坚持要提取,可以提取写作技巧（叙事结构、修辞手法、节奏控制）,但这属于另一个 skill 类型。

如果用户确认要继续,按 B 类（叙事型）参数处理。

---

### 阶段 0 — 整书理解

1. 读取用户提供的书本文本。大文件分块阅读。
2. 执行 `methodology/01-stage0-adler.md` 中的 Adler 四步 (结构 / 解释 / 批判 / 应用)。
3. 按 `templates/BOOK_OVERVIEW.md.template` 填充,写入 `books/<slug>/BOOK_OVERVIEW.md`。
4. 把产出展示给用户确认:"骨架我理解对了吗?有没有你希望重点突出的方向?" 得到确认再进入阶段 1。
5. **顺带收集**: 问用户"你最近遇到的一个具体问题是什么?" — 这个问题将作为阶段 1.5 V2 验证的测试场景来源,避免循环论证。

### 阶段 1 — 6 个 sub-agent 并行提取

**并行** spawn 6 个 Task sub-agents(使用 Agent 工具,一次调用中发起 6 个):

| sub-agent | 读取的 prompt | 产出 |
|---|---|---|
| 框架提取器 | `extractors/framework-extractor.md` | 决策框架 / 思维模型 |
| 原则提取器 | `extractors/principle-extractor.md` | 原则 / 清单 / 规则 |
| 案例提取器 | `extractors/case-extractor.md` | 作者在书中亲自使用过的实例 |
| 反例提取器 | `extractors/counter-example-extractor.md` | 书中警告的失败模式 |
| 术语提取器 | `extractors/glossary-extractor.md` | 关键概念词典 |
| 张力提取器 | `extractors/tension-extractor.md` | 内在矛盾 / 未解问题 / 未深入的反对观点 |

每个 sub-agent 独立读书、独立提取、独立输出到 `books/<slug>/candidates/<type>.md`。

#### 提取器总览（9 个）

| # | 提取器 | 阶段 | 启用条件 | 产出文件 |
|---|--------|------|---------|---------|
| 1 | `framework-extractor` | 1 | 所有类型（B 类时被 pattern-extractor 替换） | `candidates/frameworks.md` |
| 2 | `principle-extractor` | 1 | 所有类型 | `candidates/principles.md` |
| 3 | `case-extractor` | 1 | 所有类型 | `candidates/cases.md` |
| 4 | `counter-example-extractor` | 1 | 所有类型 | `candidates/counter-examples.md` |
| 5 | `glossary-extractor` | 1 | 所有类型 | `candidates/glossary.md` |
| 6 | `tension-extractor` | 1 | 所有类型 | `candidates/tensions.md` |
| 7 | `pattern-extractor` | 1 | 仅 B 类（叙事型），替换 #1 | `candidates/patterns.md` |
| 8 | `argument-chain-extractor` | 1 | 仅 C 类（理论型），额外运行 | `candidates/argument-chains.md` |
| 9 | `v3-baseline-extractor` | 1.5 | 所有类型（V3 预处理） | `candidates/v3-baselines.md` |

### 阶段 1.5 — 三重验证筛选

读取 `methodology/03-stage1.5-triple-verify.md`,执行流程:

1. **V3 基线预处理**: 运行 `extractors/v3-baseline-extractor.md`,为每条候选生成同类书对照基线
2. 对每个候选单元执行三重验证:
   - **V1 跨域**: 书中至少 2 个独立段落有佐证?
   - **V2 预测力**: 场景来源(用户问题 / 对抗性场景) + 三维度评判(可复现 / 可证伪 / 非平凡)
   - **V3 独特性**: 以基线提取器产出为锚点,同类书的回答是否有显著路径差异?

通过的进入阶段 2。不通过的写入 `books/<slug>/rejected/` 并附原因 — 保留审计轨迹,也允许用户事后捞回。

### 阶段 2 — RIA++ 构造 skill

对每个通过的单元,按 `templates/SKILL.md.template` 填充:

- **R (Reading)**: 原文引用 2-4 个片段,每段 ≤150 字,总量 ≤500 字（理论型 800 字）
- **I (Interpretation)**: 用自己的话重写方法论骨架 (避免照搬译本)
- **A1 (Past Application)**: 书中作者用过的案例
- **A2 (Future Trigger)** ★: 用户在什么情境下会需要这个 → skill 的 `description` 字段
- **E (Execution)**: 1-2-3 可执行步骤
- **B (Boundary)**: 什么时候不适用 / 来自阶段 0 批判阶段的作者盲点 / 来自 tension-extractor 的内在张力 / 来自外部批评扫描的已知反对意见

细则见 `methodology/04-stage2-ria-plus.md`。

### 阶段 3 — Zettelkasten 链接

按 `methodology/05-stage3-zettelkasten.md`:
1. 找出 skill 之间的引用关系 (A 依赖 B / A 对比 B / A 组合 B)
2. 在每个 SKILL.md 末尾补"相关 skills"段
3. 按 `templates/INDEX.md.template` 生成 `INDEX.md` (含引用图 mermaid)

### 阶段 4 — 压力测试 (darwin 兼容)

对每个 skill 按 `methodology/06-stage4-pressure-test.md`:
1. 设计 5–10 条测试 prompt,按 `templates/test-prompts.json.template` 写入 `test-prompts.json`
2. 至少包括 3 类: **应调用** / **不应调用 (诱饵)** / **边界模糊**
3. 本地跑一遍,**未过的回炉重做阶段 2** — 不做"表面修补"
4. 全部通过后通知用户: "已完成,可一键喂给 darwin-skill 自动进化"

### 阶段 5 — 反馈整合 (可选,长期运行时触发)

当 skill 被实际使用一段时间后,收集使用信号并迭代改进:

1. 在每个 skill 目录下创建 `feedback.json`（按 `templates/feedback.json.template` 格式）
2. 收集用户反馈,记录三种信号:
   - **correct**: 正确触发,输出质量好
   - **missed**: 应该触发但没触发
   - **false_alarm**: 不该触发但触发了（误激活）
3. 当 feedback 条数 ≥5 时,分析模式并决定改进方向:
   - `false_alarm` 多 → 收紧 A2 的 trigger 描述,增加排除条件
   - `missed` 多 → 放宽 A2,增加新的 trigger 信号
   - `output_quality: poor` 多 → 检查 E 段步骤是否不够具体,重写执行流程
4. 更新 SKILL.md,版本号 +1,记录改进原因

**注意**: 阶段 5 不是蒸馏流程的一部分,而是蒸馏完成后的运维流程。它与 darwin-skill 的自动进化互补:
- darwin-skill: 基于 test-prompts.json 做自动 ratcheting 进化
- 阶段 5: 基于真实用户反馈做人工迭代改进

## 质量红线 (违反则阻止输出)

1. 每个 skill 必须通过**全部**三重验证
2. 每个 skill 必须有完整的 R / I / A1 / A2 / E / B 六段
3. 原文引用 2-4 个片段,每段 ≤150 字,总量 ≤500 字（理论型 800 字）
4. 每个 skill 必须有 `test-prompts.json`,且包含诱饵测试 (不应调用的场景)
5. `description` 字段必须明确 trigger 条件,不能只是"一个关于 X 的 skill"

## 与 nuwa-skill / darwin-skill 的生态定位

- **nuwa-skill**: 蒸馏人 (思维方式 / 表达 DNA)
- **book2skill** (本 skill): 蒸馏书 (方法论 / 框架 / 原则)
- **darwin-skill**: 进化任意 skill

三者咬合: 本 skill 输出的 `test-prompts.json` 严格遵循 darwin-skill 格式,以便产出的 skill 可直接接入 darwin 做自动进化。

## 调用惯例

- **永远先试点 1 本** — 除非用户明确说"批量"
- **阶段之间主动汇报进度** — 不要静默跑完再 dump 结果
- **不凭记忆拆书** — 没文本就停下来问
- **保留审计轨迹** — candidates/ 和 rejected/ 都要留
