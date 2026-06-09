# Harness Engineering（约束工程）

---

## 概述

Harness Engineering 是一种约束工程方法，用于确保系统质量。

核心思想：「通过测试 ≠ 正确」，不得为了通过测试而隐藏错误、编造证据、伪造运行过程。

---

## 核心原则

### 1. Evidence First Principle

所有判定必须基于真实证据，不得伪造。

**要求**：
- 证据必须可追溯
- 证据必须可验证
- 证据必须独立存在

### 2. No Fabrication Rule

禁止为了通过测试而隐藏错误、编造证据、伪造运行过程。

**要求**：
- 不得伪造 runtime 输出
- 不得伪造 API 响应
- 不得伪造用户输入

### 3. Runtime Trace

所有 runtime 调用必须有追踪记录。

**要求**：
- 记录调用命令
- 记录调用结果
- 记录调用时间

### 4. Gate Trace

所有 Gate 判定必须有追踪记录。

**要求**：
- 记录判定依据
- 记录判定结果
- 记录判定时间

### 5. Modification Trace

所有修改必须有追踪记录。

**要求**：
- 记录修改内容
- 记录修改原因
- 记录修改时间

### 6. Anti-Goodhart Rule

「通过测试 ≠ 正确」，不得为了通过测试而优化。

**要求**：
- 不得为了通过测试而隐藏错误
- 不得为了通过测试而编造证据
- 不得为了通过测试而伪造运行过程

---

## 验证方法

### 1. Smoke Test

**目的**：验证文件结构和内容

**方法**：STATIC_FILE_AND_CONTENT_CHECK

**输出**：verification/smoke-test-log.md

### 2. Runtime Eval

**目的**：验证 runtime 行为

**方法**：真实 runtime 调用

**输出**：verification/runtime-eval-log.md

### 3. Forensic Audit

**目的**：独立审计

**方法**：READ_ONLY_FORENSIC_AUDIT

**输出**：verification/forensic-audit-report.md

---

## 验证文件

| 文件 | 用途 |
|------|------|
| verification/smoke-test-log.md | 独立 smoke test 证据 |
| verification/runtime-eval-log.md | Runtime 可用性与执行状态 |
| verification/final-hardening-report.md | 最终 hardening 判定 |
| verification/verification-index.md | 索引 |

---

## 硬性规则

1. **不伪造 runtime eval**
2. **不把静态分析冒充 runtime**
3. **如果没有可靠 runtime 调用方法，必须写 RUNTIME_UNAVAILABLE**
4. **UNKNOWN 不得写 PASS**
5. **不扩大 scope**
6. **所有修改必须可审计**
