---
title: RIA-TV++ 方法论对比分析
source: kangarooking/cangjie-skill (⭐1127)
analyzed: 2026-06-09
---

# RIA-TV++ vs 女娲五层蒸馏法 对比

## RIA-TV++ 六阶段流水线

cangjie-skill 的核心方法论，用于把任意书籍 PDF 蒸馏成 10-25 个独立 Agent Skills。

| 阶段 | 做什么 | 产出 |
|---|---|---|
| 0. Adler 整书理解 | 结构/解释/**批判**/应用四步 | BOOK_OVERVIEW.md |
| 1. 并行提取 | 5个sub-agent（框架/原则/案例/反例/术语） | candidates/ |
| 1.5. 三重验证 | V1跨域 + V2预测力 + V3独特性 | 通过率25-50% |
| 2. RIA++ 构造 | R原文+I自述+A1案例+A2触发+E步骤+B边界 | SKILL.md |
| 3. Zettelkasten链接 | skill间依赖/对比/组合关系 | INDEX.md |
| 4. 压力测试 | should_trigger + 诱饵 + 边界模糊 | test-prompts.json |

### 命名来源

- **RIA**：赵周《这样读书就够了》便签拆书法（Reading / Interpretation / Appropriation）
- **TV**：Triple Verification，三重验证
- **++**：面向 agent 执行的扩展（E Execution + B Boundary）

### 思想来源

| 来源 | 借鉴内容 |
|---|---|
| Mortimer Adler《如何阅读一本书》 | 阶段0分析阅读三阶段 |
| 赵周 RIA 拆书法 | 阶段2 R-I-A1-A2 骨架，A2→trigger |
| Niklas Luhmann Zettelkasten | 原子化+链接+用自己的话重写 |
| Tiago Forte Progressive Summarization | 可验证压缩链条 |
| nuwa-skill | 并行extractor+三重验证 |
| darwin-skill | test-prompts.json格式+可进化性 |

## 核心差异

| 维度 | 女娲五层（ours） | RIA-TV++（cangjie） |
|---|---|---|
| 蒸馏维度 | 表达DNA→心智模型→决策启发式→反模式→诚实边界 | R原文→I自述→A1案例→A2触发→E步骤→B边界 |
| **独有优势** | Reading Gate（证明读过） | 三重验证（跨域+预测力+独特性） |
| 粒度 | 1书→1个skill（5-9个模型） | 1书→10-25个独立skill |
| 验证 | Smoke+Runtime+Forensic | 诱饵测试+darwin兼容 |
| 输入源 | 微信读书标注（有证据链） | 任意PDF（无阅读证据） |
| 平台 | Hermes专用 | Claude Code + OpenClaw |

## 值得借鉴的 3 个点

### 1. 三重验证筛选（V1/V2/V3）

Reading Gate 验证"读过"，但没有验证"提取的内容值不值得做skill"。三重验证补这个缺口：

- **V1 跨域**：该方法论在书中至少 2 个独立语境有佐证（不是同一案例换说法）
- **V2 预测力**：能用它推导出书里没明说的问题答案？只复述书中例子 = 没有解释力
- **V3 独特性**：不是任何聪明人都会说的常识？抹掉作者名字还能说出来 = 不通过

通过率预期 25-50%。未通过的降级为 example/引用，不独立成 skill。

**集成建议**：在蒸馏流程中，在女娲五层蒸馏之前增加三重验证筛选步骤。

### 2. test-prompts.json 诱饵测试

当前有 smoke test + runtime eval + forensic audit，都是验证"skill 内容质量"，缺少**触发精准度验证**。

为每个 book-skill 生成 `test-prompts.json`，含三类测试：
- `should_trigger` 3-5 条（该调用时是否调用）
- `should_not_trigger` 2-3 条（诱饵，不该调用时是否忍住）— **没有诱饵的 skill 一律打回**
- `edge_case` 1-3 条（边界模糊场景判断）

格式兼容 darwin-skill，可直接接入自动进化。

**集成建议**：在 eval 阶段增加 trigger precision eval，输出 test-prompts.json。

### 3. A2 触发条件精确化

RIA-TV++ 的 A2 字段要求：
1. 用户在什么情境下遇到这类问题（3-5条场景描述）
2. 这些情境的语言信号是什么（用户会说什么样的话）
3. 和哪些相邻 skill 不同（避免互相抢调用）

A2 的产出直接写入 skill frontmatter 的 `description` 字段。

**集成建议**：book-skill-template 的触发词设计参考 A2 的三层精确化。

## 不需要借鉴的

- **多skill拆分粒度**：女娲的单skill+5-9模型设计更紧凑，适合阅读场景
- **Adler整书理解**：有微信读书标注作为天然的阅读理解证据，不需要重复证明
- **并行sub-agent提取**：微信读书标注本身就是用户筛选过的内容，不需要5个提取器

## 相关 Issue

- [panglihaoshuai/hermes-weread-skill-factory#1](https://github.com/panglihaoshuai/hermes-weread-skill-factory/issues/1) — 引入 test-prompts.json 诱饵测试 + RIA-TV++ 三重验证
