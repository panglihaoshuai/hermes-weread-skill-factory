# Skill State Machine（状态机）

## 概述

weread-to-skill-factory 是一个有限状态机（FSM）。

一旦进入流程，必须始终处于某个明确状态。不允许处于"无状态"或"自由聊天"状态。

---

## 允许状态（15 个）

| # | 状态 | 含义 |
|---|------|------|
| 0 | IDLE | 初始状态，未选择书 |
| 1 | BOOK_SELECTED | 用户选择书，待 Gate |
| 2 | GATE_RUNNING | Reading Gate 执行中 |
| 3 | GATE_COMPLETED | Gate 评分完成 |
| 4 | GUIDED_VIVA_RUNNING | Guided Viva 进行中 |
| 5 | GUIDED_VIVA_COMPLETED | Q10 完成，待评分 |
| 6 | POST_VIVA_SCORING | 评分执行中 |
| 7 | WAITING_WRITE_APPROVAL | 等待用户批准生成 Draft |
| 8 | DRAFT_BOOK_SKILL_GENERATING | Draft 生成中 |
| 9 | DRAFT_BOOK_SKILL_COMPLETED | Draft 完成，待 eval |
| 10 | EVAL_RUNNING | eval 执行中 |
| 11 | EVAL_COMPLETED | eval 完成，待安装批准 |
| 12 | WAITING_INSTALL_APPROVAL | 等待用户批准安装 |
| 13 | BOOK_SKILL_INSTALLED | 正式安装完成 |
| 14 | TERMINATED | 合法退出 |

---

## 状态流转图

```
IDLE
  │
  ▼
BOOK_SELECTED
  │
  ▼
GATE_RUNNING
  │
  ▼
GATE_COMPLETED
  │
  ├──────────────────────────────────────┐
  │                                      │
  ▼ (需要 Viva)                          ▼ (不需要 Viva)
GUIDED_VIVA_RUNNING                     WAITING_WRITE_APPROVAL
  │                                      │
  ▼                                      │
GUIDED_VIVA_COMPLETED                    │
  │                                      │
  ▼                                      │
POST_VIVA_SCORING                        │
  │                                      │
  ▼                                      │
WAITING_WRITE_APPROVAL ◄─────────────────┘
  │
  ▼
DRAFT_BOOK_SKILL_GENERATING
  │
  ▼
DRAFT_BOOK_SKILL_COMPLETED
  │
  ▼
EVAL_RUNNING
  │
  ▼
EVAL_COMPLETED
  │
  ▼
WAITING_INSTALL_APPROVAL
  │
  ▼
BOOK_SKILL_INSTALLED
  │
  ▼
TERMINATED
```

---

## 状态定义

### IDLE

- **Entry 条件**：初始状态，或流程被取消后重置
- **Exit 条件**：用户选择书
- **Next State**：BOOK_SELECTED
- **Forbidden State**：不得跳到 GATE_RUNNING 或之后

### BOOK_SELECTED

- **Entry 条件**：用户选择书
- **Exit 条件**：开始 Reading Gate
- **Next State**：GATE_RUNNING
- **Forbidden State**：不得跳到 GATE_COMPLETED 或之后

### GATE_RUNNING

- **Entry 条件**：开始 Reading Gate
- **Exit 条件**：Gate 评分完成
- **Next State**：GATE_COMPLETED
- **Forbidden State**：不得跳到 GUIDED_VIVA_RUNNING 或之后

### GATE_COMPLETED

- **Entry 条件**：Gate 评分完成
- **Exit 条件**：判断是否需要 Viva
- **Next State**：GUIDED_VIVA_RUNNING 或 WAITING_WRITE_APPROVAL
- **Forbidden State**：不得跳到 POST_VIVA_SCORING 或之后（除非经过 Viva）

### GUIDED_VIVA_RUNNING

- **Entry 条件**：进入 Guided Viva
- **Exit 条件**：Q10 完成
- **Next State**：GUIDED_VIVA_COMPLETED
- **Forbidden State**：不得跳到 POST_VIVA_SCORING 或之后（除非 Q10 完成）

### GUIDED_VIVA_COMPLETED

- **Entry 条件**：Q10 完成
- **Exit 条件**：开始评分
- **Next State**：POST_VIVA_SCORING
- **Forbidden State**：不得跳到 WAITING_WRITE_APPROVAL 或之后（除非评分完成）

### POST_VIVA_SCORING

- **Entry 条件**：开始评分
- **Exit 条件**：评分完成
- **Next State**：WAITING_WRITE_APPROVAL
- **Forbidden State**：不得跳到 DRAFT_BOOK_SKILL_GENERATING 或之后

### WAITING_WRITE_APPROVAL

- **Entry 条件**：评分完成，等待用户批准
- **Exit 条件**：用户批准
- **Next State**：DRAFT_BOOK_SKILL_GENERATING
- **Forbidden State**：不得跳到 DRAFT_BOOK_SKILL_COMPLETED 或之后（除非用户批准）

### DRAFT_BOOK_SKILL_GENERATING

- **Entry 条件**：用户批准生成 Draft
- **Exit 条件**：Draft 生成完成
- **Next State**：DRAFT_BOOK_SKILL_COMPLETED
- **Forbidden State**：不得跳到 EVAL_RUNNING 或之后（除非 Draft 完成）

### DRAFT_BOOK_SKILL_COMPLETED

- **Entry 条件**：Draft 生成完成
- **Exit 条件**：开始 eval
- **Next State**：EVAL_RUNNING
- **Forbidden State**：不得 TERMINATED（必须经过 eval）

### EVAL_RUNNING

- **Entry 条件**：开始 eval
- **Exit 条件**：eval 完成
- **Next State**：EVAL_COMPLETED
- **Forbidden State**：不得跳到 WAITING_INSTALL_APPROVAL 或之后（除非 eval 完成）

### EVAL_COMPLETED

- **Entry 条件**：eval 完成
- **Exit 条件**：等待用户批准安装
- **Next State**：WAITING_INSTALL_APPROVAL
- **Forbidden State**：不得跳到 BOOK_SKILL_INSTALLED 或之后（除非用户批准）

### WAITING_INSTALL_APPROVAL

- **Entry 条件**：eval 完成，等待用户批准安装
- **Exit 条件**：用户批准安装
- **Next State**：BOOK_SKILL_INSTALLED
- **Forbidden State**：不得 TERMINATED（除非用户批准安装或取消）

### BOOK_SKILL_INSTALLED

- **Entry 条件**：用户批准安装，安装完成
- **Exit 条件**：记录安装结果
- **Next State**：TERMINATED
- **Forbidden State**：无

### TERMINATED

- **Entry 条件**：满足终止条件之一
- **Exit 条件**：无（终态）
- **Next State**：无
- **Forbidden State**：无

---

## 终止条件（Termination Protocol）

只有满足以下条件之一，才能进入 TERMINATED：

### 情况一：用户明确取消

用户明确说「取消」「停止」「退出」「不做了」。

### 情况二：BOOK_SKILL_INSTALLED 完成

正式 book-skill 安装完成。

### 情况三：不可恢复错误

系统发生不可恢复错误，并明确记录失败原因。

---

## 终止记录

进入 TERMINATED 时必须记录：

```
Final State: TERMINATED
Exit Reason: [用户取消 / 安装完成 / 不可恢复错误]
Scored: [YES / NO]
Draft Generated: [YES / NO]
Eval Passed: [YES / NO / N/A]
Write Approved: [YES / NO / N/A]
Install Approved: [YES / NO / N/A]
Installed: [YES / NO / N/A]
```

---

## 压缩恢复检查点（Compression Recovery Checkpoint）

每轮执行后必须维护 checkpoint：

```
Current State: [状态名]
Previous State: [状态名]
Book: [书名]
Mode: [Strict Viva / Guided Viva / Fixture Viva / Gate Only]
Gate Result: [分数 / UNKNOWN / NOT RUN]
Viva Progress: [Qx/10 / NOT STARTED / COMPLETED]
Answers Completed: [0-10]
Required Next Action: [具体动作]
Write Permission: [NOT APPROVED / APPROVED]
Install Permission: [NOT APPROVED / APPROVED]
Termination Status: [NOT TERMINATED / TERMINATED]
```

---

## 压缩恢复协议

如果发生 `/compress`、context restore、长对话恢复：

1. 读取 checkpoint
2. 恢复状态机
3. 输出当前状态：
   ```
   【状态恢复】
   Current State: [状态名]
   Previous State: [状态名]
   Book: [书名]
   Required Next Action: [具体动作]
   ```
4. 继续 Required Next Action

**禁止**：
- 忘记当前阶段
- 重新开始整个流程
- 进入自由聊天

---

## 状态转换守卫（State Transition Guard）

每次输出前必须检查：

如果 `Current State ≠ TERMINATED` 且 `Required Next Action 非空`：

必须继续执行 Required Next Action。

**禁止**：
- 输出结束语
- 输出「希望对你有帮助」
- 输出泛泛建议
- 默认结束对话

只有 `Current State = TERMINATED` 时，才允许正常退出。

---

## 状态转换追踪（State Transition Trace）

每次状态变化必须输出：

```
Previous State: [状态名]
Trigger: [触发原因]
Evidence: [证据]
New State: [状态名]
```

禁止隐式跳转。
