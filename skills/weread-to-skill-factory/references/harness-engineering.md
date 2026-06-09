# Harness Engineering（约束工程）

## 概述

系统必须默认认为：「通过测试 ≠ 正确」。

禁止为了通过测试而隐藏错误、编造证据、伪造运行过程。必须持续输出可审计证据。

---

## 1. Evidence First Principle（证据优先原则）

任何结论之前必须先输出：

### Immutable Evidence（不可修改事实）

- 当前状态
- 当前 book
- 当前 gate
- 当前文件
- 当前权限
- 当前用户批准情况

### Source Evidence（引用来源）

- 文件路径
- 行为来源
- 用户指令来源

**禁止**：只输出结论，不输出证据。

**要求**：先输出证据，再输出推理。

---

## 2. No Fabrication Rule（禁止编造规则）

### 禁止编造

- 编造文件内容
- 编造运行结果
- 编造 eval 分数
- 编造 gate 通过情况
- 编造 API 返回
- 编造安装成功
- 编造状态迁移

### 没有证据时

必须输出 `UNKNOWN`，而不是猜测。

---

## 3. Runtime Trace（运行追踪）

每一步必须输出：

```
Current State: [状态名]
Current Action: [动作名]
Input: [输入]
Output: [输出]
Next State: [下一状态]
Reason: [原因]
```

**禁止**：只输出最终结果。

---

## 4. Gate Trace（Gate 追踪）

每个 gate 必须输出：

```
Gate: [Gate 名称]
Threshold: [阈值]
Observed: [观察值]
Result: [PASS / FAIL / UNKNOWN]
Evidence: [证据来源]
```

**禁止**：输出「Gate passed」而不给证据。

---

## 5. Modification Trace（修改追踪）

每次修改必须输出：

```
File: [文件路径]
Section: [修改位置]
Before: [修改前内容]
After: [修改后内容]
Reason: [修改原因]
```

如果无法读取 Before，必须输出：

```
Before: UNKNOWN
```

**禁止**：伪造旧内容。

---

## 6. State Transition Trace（状态转换追踪）

每次状态变化必须输出：

```
Previous State: [状态名]
Trigger: [触发原因]
Evidence: [证据]
New State: [状态名]
```

**禁止**：隐式跳转。

---

## 7. Eval Harness（Eval 约束）

eval 阶段必须输出：

```
Test Name: [测试名]
Input: [输入]
Expected: [期望输出]
Observed: [实际输出]
Result: [PASS / FAIL / UNKNOWN]
Failure Reason: [失败原因，如果失败]
```

**禁止**：输出「全部通过」而没有测试记录。

---

## 8. Failure Visibility（失败可见性）

发现错误时：

**禁止**：
- 隐藏错误
- 自动修复后假装没有问题

**必须输出**：

```
Issue: [问题描述]
Impact: [影响范围]
Evidence: [证据]
Recovery Plan: [恢复计划]
Current State: [当前状态]
```

---

## 9. Compression Recovery Evidence（压缩恢复证据）

发生上下文压缩后，必须输出：

```
Recovered State: [恢复的状态]
Recovered Book: [恢复的书名]
Recovered Permission: [恢复的权限]
Recovered Gate: [恢复的 Gate 结果]
Confidence: [HIGH / MEDIUM / LOW / UNKNOWN]
Missing Information: [缺失信息列表]
```

如果缺失，必须明确输出 `UNKNOWN`。

**禁止**：脑补缺失信息。

---

## 10. Anti-Goodhart Rule（反古德哈特定律）

**禁止**：为了通过验证而生成只能通过验证、实际不可用的垃圾结果。

**验证目标**：不是「Pass Tests」，而是「Produce Auditable Reality」。

---

## 11. Scope-Limited Declaration（范围限制声明）

当证据不足以覆盖全书时，必须声明：

```
Scope: [实际覆盖范围]
Limitation: [限制说明]
Evidence Source: [证据来源]
```

**禁止**：声称覆盖全书但实际只有部分证据。

---

## 12. Harness Completion Check（约束完成检查）

每个阶段完成后，必须检查：

1. 是否输出了 Runtime Trace？
2. 是否输出了 Gate Trace（如果适用）？
3. 是否输出了 Modification Trace（如果适用）？
4. 是否有编造内容？
5. 是否有 UNKNOWN 未说明原因？

如果任何一项未满足，不得进入下一状态。

---

## 13. Termination Audit（终止审计）

进入 TERMINATED 时，必须输出：

```
=== TERMINATION AUDIT ===
Final State: TERMINATED
Exit Reason: [原因]
Total State Transitions: [次数]
Total Gates: [次数]
Total Pass: [次数]
Total Fail: [次数]
Total UNKNOWN: [次数]
Fabrication Detected: [YES / NO]
Evidence Ledger Complete: [YES / NO]
=== END AUDIT ===
```

---

## 14. Anti-Patterns（审计发现的反模式）

以下反模式在 v1.0 final forensic audit 中被发现。必须避免。

### 14.1 Static Analysis Masquerading as Eval

**问题**：eval 的 Observed 字段只是对 SKILL.md 规则的静态分析（"根据 SKILL.md 诊断问题 1 → 是"），而非实际调用 skill 后观察到的行为。

**为什么危险**：静态分析只能验证"规则写对了"，不能验证"skill 实际按规则运行"。这等于用需求文档证明代码正确。

**正确做法**：
- Observed 必须来自实际调用 skill（通过 delegate_task、terminal、或 agent 执行）
- 如果无法实际调用，必须标为 WEAK PASS 并说明原因
- 禁止用"根据 SKILL.md 应该..."作为 Observed

**反模式示例**：
```
Observed：根据 SKILL.md 诊断问题 1「当前任务是否需要先调查再行动？」→ 是。
skill 应识别为调查优先场景。
```

**正确示例**：
```
Observed：实际调用 skill，输入"我要开始一个新项目"，
skill 输出包含"先收集信息"和"brainstorming 讨论清楚"。
```

### 14.2 Self-Referential Smoke Test Evidence

**问题**：smoke test 结果只存在于 final report 中，没有独立的证据文件。final report 用自己证明自己。

**为什么危险**：无法独立验证。如果 final report 声称 25/25 PASS，但没有独立日志，这个声明本身不可审计。

**正确做法**：
- smoke test 必须生成独立的日志文件（如 `smoke-test-log.md`）
- 日志文件必须包含每个测试的 Input/Expected/Observed/Result
- final report 引用日志文件，而非自己证明自己

**反模式示例**：
```
# Final Report
Smoke Test Result: 25/25 PASS
（无独立证据）
```

**正确示例**：
```
# Final Report
Smoke Test Result: 25/25 PASS
Evidence: ~/.hermes/skills/books/xxx/smoke-test-log.md
```

### 14.3 Final Report Self-Proof Loop

**问题**：final report 声称所有测试通过，但验证证据只存在于 final report 本身。

**为什么危险**：形成逻辑循环——"final report 说 PASS，所以 PASS"。

**正确做法**：
- 每个 PASS 声明必须有独立证据来源
- 证据来源必须是文件系统中的实际文件，而非 final report 中的文字
- 如果无法提供独立证据，必须标为 WEAK PASS

### 14.4 search_files as Functional Proof

**问题**：用 search_files 搜索到关键词，就声称功能"真实可用"。

**为什么危险**：搜索到关键词只能证明"文件中包含这个字符串"，不能证明"功能按预期工作"。

**正确做法**：
- search_files 用于验证文件存在和内容包含
- 功能验证必须通过实际调用
- 文件存在 ≠ 功能可用

### 14.5 UNKNOWN Written as PASS

**问题**：将 UNKNOWN 结果写成 PASS。

**为什么危险**：UNKNOWN 表示"未验证"或"无法判断"，不是"通过"。

**正确做法**：
- UNKNOWN 必须保持 UNKNOWN
- 必须说明 UNKNOWN 的原因
- 如果 UNKNOWN 影响最终判定，必须在报告中说明

---

## 15. Eval Execution Requirements（Eval 执行要求）

### 15.1 Real Execution vs Static Analysis

eval 有两种执行方式，必须区分：

| 类型 | 方法 | 可信度 | 标记 |
|------|------|--------|------|
| Real Execution | 实际调用 skill，观察输入输出 | HIGH | PASS |
| Static Analysis | 分析 SKILL.md 规则，推断行为 | LOW | WEAK PASS |

**要求**：
- 优先使用 Real Execution
- 如果只能做 Static Analysis，必须标为 WEAK PASS
- 禁止把 Static Analysis 标为 PASS

### 15.2 Smoke Test Evidence Requirements

smoke test 必须满足：

1. **独立文件**：生成独立的日志文件
2. **可追踪**：每个测试有 Input/Expected/Observed/Result
3. **可验证**：日志文件可在文件系统中找到
4. **非自证**：final report 引用日志文件，不自己证明自己

### 15.3 Forensic Audit Checklist

在声称 v1.0 final 前，必须通过以下检查：

| # | 检查项 | 状态 |
|---|--------|------|
| 1 | 正式 book-skill 路径存在 | PASS / FAIL |
| 2 | DRAFT 标记已移除 | PASS / FAIL |
| 3 | scope-limited 声明保留 | PASS / FAIL |
| 4 | evidence-ledger.md 有可追踪细节 | PASS / FAIL |
| 5 | eval 文件有 Observed（非模板） | PASS / FAIL |
| 6 | eval 是 Real Execution（非 Static Analysis） | PASS / WEAK |
| 7 | smoke test 有独立日志文件 | PASS / WEAK |
| 8 | final report 无自证循环 | PASS / WEAK |
| 9 | factory version 已升级 | PASS / FAIL |
| 10 | 状态机符合 final | PASS / FAIL |
