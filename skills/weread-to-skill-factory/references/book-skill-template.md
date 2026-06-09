# book-skill 文件模板

## Level 0: Reject 输出模板

```
# 《书名》蒸馏拒绝

**原因**：[具体原因，如「无个人阅读证据」「口试未通过」]
**建议**：[阅读路径建议]
```

## Level 1: Reading Card 模板

```markdown
# 《书名》Reading Card

> 生成日期：YYYY-MM-DD
> Gate 判定：Level 1（证据不足，仅生成概览）

## 一句话总论
[基于热门划线和简介的概括]

## 核心问题
[这本书试图解决什么问题]

## 章节结构地图
[基于章节目录的逻辑梳理]

## 推荐阅读章节
[基于热门划线密度推荐优先阅读的章节]

## 推荐划线 Top 5
[热门划线中最值得思考的 5 条]

## 为什么不生成 skill
[具体说明证据不足在哪里]

## 如何升级为 skill
[需要用户做什么：精读后回来，或通过口试]
```

## Level 2: Draft Book Skill 模板

```yaml
---
name: <book-slug>-skill
description: >
  [DRAFT] 蒸馏自《书名》的认知框架。需要补充证据后才能正式启用。
tags: [book, <category>, draft]
---
```

```markdown
# 《书名》认知框架 [DRAFT]

> ⚠️ 这是草稿版本，需要补充证据后才能正式启用。
> Gate 判定：Level 2（条件通过）
> 待补充：[具体说明需要什么]

## 核心身份
"你是《书名》的认知操作系统..."

## 核心模型（初步）
[基于现有证据的初步提取，标注证据等级]

## 待验证
[需要用户补充确认的内容]
```

## Level 3: Installed Book Skill 模板

```yaml
---
name: <book-slug>-skill
description: >
  蒸馏自《书名》的认知框架。用[作者]的思维方式帮用户分析[领域]问题。
  触发词：「用XX框架分析」「XX会怎么看」「XX的方法论」。
  不适用：[场景1]、[场景2]。
tags: [book, <category>]
---
```

```markdown
# 《书名》认知框架

> 蒸馏自微信读书个人划线 N 条 + 批注 M 条 + 热门划线 20 条
> 蒸馏日期：YYYY-MM-DD
> Gate 路径：WeRead Evidence / Private Reading Viva
> Gate 分数：X/20 或 X/30
> Gate 判定：Level 3（强通过）
> 证据等级：A/B/C/D/E/F

## 核心身份
"你是《书名》的认知操作系统。当用户遇到[领域]问题时，用[作者]的思维框架帮他们分析。"

## 一句话总论
[核心认知操作系统]

## 核心模型

### 模型1: <名称>
- **一句话**：[核心思想]
- **适用场景**：[什么时候用]
- **诊断问题**：[用这个模型时该问什么]
- **反模式**：[不要做什么]
- **来源**：[证据等级 + 原文引用]

（5-9 个模型）

## 决策启发式

| 场景 | 启发式规则 | 来源 |
|------|-----------|------|
| [场景] | [if-then 规则] | [证据等级] |

## 行动协议

当用户遇到 [具体场景] 时：
1. [步骤1]
2. [步骤2]
3. [步骤3]

## 反模式（绝对不要）
- 不要...
- 不要...

## 诊断问题清单
1. [问题1]
2. [问题2]

## 诚实边界
- 擅长：...
- 不擅长：...
- 不适用场景：...

## 与用户项目的连接
- [项目1]：...
- [项目2]：...
```

## references/ 目录结构

### Level 3 完整结构
```
~/.hermes/skills/books/<book-slug>-skill/
├── SKILL.md
└── references/
    ├── book-map.md
    ├── distilled-framework.md
    ├── decision-rules.md
    ├── anti-patterns.md
    ├── user-highlights.md
    ├── user-notes.md
    ├── evidence-ledger.md
    └── limits.md
```

### Level 2 精简结构
```
~/.hermes/skills/books/<book-slug>-skill/
├── SKILL.md                    ← 标注 [DRAFT]
└── references/
    ├── evidence-ledger.md
    └── limits.md
```

### Level 1 只有 reading card
```
~/.hermes/reading-cards/<book-slug>.md
```
