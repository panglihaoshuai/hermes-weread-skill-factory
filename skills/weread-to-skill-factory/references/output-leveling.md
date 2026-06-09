# 产物分级机制（Output Leveling）

根据 Reading Gate 结果，输出分为四级。

## Level 0: Reject

**不生成任何 skill。** 只输出：
- 拒绝原因（具体说明哪条 Gate 规则未通过）
- 建议阅读路径（「建议先精读再回来蒸馏」）

## Level 1: Reading Card

**适用于**：用户感兴趣但无精读证据。

输出路径：`~/.hermes/reading-cards/<book-slug>.md`

内容：
- 一句话总论（基于热门划线和简介的概括）
- 核心问题
- 章节结构地图
- 推荐阅读章节（基于热门划线密度）
- 推荐划线 Top 5
- 为什么不生成 skill（具体说明证据不足在哪里）
- 如何升级为 skill（需要用户做什么）

**不安装，不进入 skill 系统。**

## Level 2: Draft Book Skill

**适用于**：读过但证据不足、理解尚可、迁移场景不够清晰。

输出路径：`~/.hermes/skills/books/<book-slug>-skill/`
```
├── SKILL.md                    ← 标注为 [DRAFT]
└── references/
    ├── evidence-ledger.md
    └── limits.md
```

**不默认启用。** 必须要求用户补充证据或口试后才能升级为 Level 3。

## Level 3: Installed Book Skill

**适用于**：真正精读、有个人判断、有现实迁移、有可操作模型的书。

输出路径：`~/.hermes/skills/books/<book-slug>-skill/`
```
├── SKILL.md                    ← 核心认知框架 (< 5KB)
└── references/
    ├── book-map.md             ← 章节结构地图
    ├── distilled-framework.md  ← 蒸馏框架详情
    ├── decision-rules.md       ← 决策启发式
    ├── anti-patterns.md        ← 反模式
    ├── user-highlights.md      ← 你的划线（原文）
    ├── user-notes.md           ← 你的批注（原文）
    ├── evidence-ledger.md      ← 证据台账
    └── limits.md               ← 诚实边界 + 不适用场景
```

**必须包含 evals：**
- `evals/trigger-tests.md`
- `evals/application-tests.md`
- `evals/adversarial-tests.md`
