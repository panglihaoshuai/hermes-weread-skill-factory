# Architecture（架构）

---

## 概述

weread-to-skill-factory 是一个本地 Hermes skill factory，用于把微信读书中精读过的书蒸馏成可复用的 book-skill。

---

## 核心组件

### 1. Reading Gate（精读资格判定）

**功能**：判定用户是否真正精读过某本书

**判定路径**：
- 路径 A：WeRead 证据路径（有个人划线/批注/完成度）
- 路径 B：私下精读自证路径（无个人证据 → Private Reading Viva 口试）

**评分机制**：
- WeRead 证据：20 分制（5 维度）
- Viva 口试：30 分制

**产物分级**：
- Level 0：Reject（不生成 skill）
- Level 1：Reading Card（概览，不安装）
- Level 2：Draft Book Skill（草稿，不默认启用）
- Level 3：Installed Book Skill（正式安装，必须有 evals）

### 2. Skill Factory（技能工厂）

**功能**：把精读过的书蒸馏成 book-skill

**蒸馏方法**：女娲五层蒸馏法
- 表达 DNA → 心智模型 → 决策启发式 → 反模式 → 诚实边界

**输出**：
- 一句话总论
- 章节结构地图
- 5-9 个核心模型
- 决策启发式
- 反模式
- 诊断问题
- 行动协议
- 诚实边界

### 3. Verification（验证系统）

**功能**：确保 book-skill 质量

**验证方法**：
- Smoke test：文件结构和内容检查
- Runtime eval：真实 runtime 调用验证
- Forensic audit：独立审计

**验证文件**：
- verification/smoke-test-log.md
- verification/runtime-eval-log.md
- verification/final-hardening-report.md
- verification/verification-index.md

---

## 数据流

```
用户请求蒸馏某本书
    ↓
Reading Gate 检查
    ├─ WeRead 证据路径
    └─ Private Reading Viva 路径
    ↓
评分判定 → Level 0/1/2/3
    ↓
Skill Factory 蒸馏
    ↓
Verification 验证
    ↓
安装 book-skill
```

---

## 约束工程

### Evidence First Principle

所有判定必须基于真实证据，不得伪造。

### No Fabrication Rule

禁止为了通过测试而隐藏错误、编造证据、伪造运行过程。

### Runtime Trace

所有 runtime 调用必须有追踪记录。

### Gate Trace

所有 Gate 判定必须有追踪记录。

### Anti-Goodhart Rule

「通过测试 ≠ 正确」，不得为了通过测试而优化。

---

## 状态机

15 个允许状态：

```
IDLE → BOOK_SELECTED → GATE_RUNNING → GATE_COMPLETED
  → GUIDED_VIVA_RUNNING → GUIDED_VIVA_COMPLETED → POST_VIVA_SCORING
  → WAITING_WRITE_APPROVAL → DRAFT_BOOK_SKILL_GENERATING → DRAFT_BOOK_SKILL_COMPLETED
  → EVIDENCE_BACKFILL → EVAL_RUNNING → EVAL_COMPLETED
  → WAITING_INSTALL_APPROVAL → BOOK_SKILL_INSTALLED → TERMINATED
```

---

## 文件结构

```
book-skill/
├── SKILL.md
├── references/
│   ├── book-map.md
│   ├── distilled-framework.md
│   ├── decision-rules.md
│   ├── anti-patterns.md
│   ├── diagnostic-questions.md
│   ├── action-protocols.md
│   ├── user-highlights.md
│   ├── user-notes.md
│   ├── evidence-ledger.md
│   └── limits.md
├── evals/
│   ├── trigger-tests.md
│   ├── application-tests.md
│   └── adversarial-tests.md
└── verification/
    ├── smoke-test-log.md
    ├── runtime-eval-log.md
    ├── final-hardening-report.md
    └── verification-index.md
```
