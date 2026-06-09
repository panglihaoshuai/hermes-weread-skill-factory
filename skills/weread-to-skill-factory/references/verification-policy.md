# Verification Policy（验证政策）

所有检查必须输出明确状态，不得模糊通过。

## 状态定义

| 状态 | 含义 | 使用场景 |
|------|------|---------|
| PASS | 检查通过，有证据 | 执行成功且输出符合预期 |
| FAIL | 检查未通过 | 执行失败或输出不符合预期 |
| UNKNOWN | 未执行或结果不可判断 | API 未验证、数据不可用、无法确认 |
| SKIPPED | 跳过（有合理理由） | 不适用、用户明确跳过、依赖缺失 |

## 硬性规则

以下情况**一律不得写 PASS**：

1. **exit 1** — 进程异常退出
2. **timeout** — 执行超时
3. **blocked** — 被阻塞（权限、依赖、网络等）
4. **user denied** — 用户拒绝
5. **output truncated** — 输出被截断
6. **未执行的检查** — 必须标 UNKNOWN 或 SKIPPED

## Clarify / Approval / Confirmation 超时处理

如果 clarify、approval、confirmation 等待用户响应时超时：
- **不得自行决定继续**
- **必须停止**，不得执行 write_file / skill_manage / install / delete
- 状态标记为 **BLOCKED** 或 **NEEDS_USER_CONFIRMATION**
- 不得把该项写成 PASS
- 下一轮必须重新请求用户确认

## 运行模式

本 skill 有三种运行模式，由用户显式选择或由上下文推断：

| 模式 | 调用 API | 写文件 | 安装 skill | 用途 |
|------|---------|--------|-----------|------|
| **Fixture Dry-run** | ❌ 禁止 | ❌ 禁止 | ❌ 禁止 | 用模拟数据测试 Gate 规则逻辑 |
| **Live Gate Dry-run** | ✅ 允许 | ❌ 禁止 | ❌ 禁止 | 用真实 API 测试 Gate 判定，但不产生任何副作用 |
| **Write-approved Run** | ✅ 允许 | ✅ 用户批准后允许 | ✅ 用户批准后允许 | 正式蒸馏，必须先通过 Gate |

### 模式选择规则

1. 用户说「dry-run」「测试」「模拟」→ 默认 **Fixture Dry-run**
2. 用户说「用真实数据测」「调 API 但别写」→ **Live Gate Dry-run**
3. 用户说「开始蒸馏」「批准」「执行」→ **Write-approved Run**
4. 不确定时，默认 **Fixture Dry-run**（最安全）

### 模式违规处理

- 在 Fixture Dry-run 中调用了 API → 立即停止，标记 FAIL
- 在任何 Dry-run 中写了文件 → 立即停止，标记 FAIL，回滚
- 在 Write-approved Run 中未获批准就写文件 → 立即停止，标记 FAIL

## API UNKNOWN 处理

当 WeRead API 返回错误、超时或未测试时：
- 记为 UNKNOWN，不得记为 0 分
- UNKNOWN 不等于 FAIL
- UNKNOWN 时必须标注原因（「API 未测试」「API 返回错误」「网络超时」）
- 如果 UNKNOWN 影响 Gate 评分，必须在报告中说明

## 报告要求

最终报告必须包含：
1. 检查项名称
2. 检查方法/命令
3. 实际结果（含证据片段）
4. 状态：PASS / FAIL / UNKNOWN / SKIPPED
5. 失败原因（如果是 FAIL）

---

## 状态机验证检查表

修改 weread-to-skill-factory 后，必须逐项验证：

| # | 检查项 | 状态 |
|---|--------|------|
| 1 | `references/skill-state-machine.md` 已创建 | PASS / FAIL |
| 2 | `references/harness-engineering.md` 已创建 | PASS / FAIL |
| 3 | SKILL.md 只保留简短引用，没有塞入长状态机 | PASS / FAIL |
| 4 | 状态机包含 15 个状态（IDLE 到 TERMINATED） | PASS / FAIL |
| 5 | 状态机包含 EVAL_RUNNING / EVAL_COMPLETED / WAITING_INSTALL_APPROVAL / BOOK_SKILL_INSTALLED | PASS / FAIL |
| 6 | DRAFT_BOOK_SKILL_COMPLETED 不再直接 TERMINATED | PASS / FAIL |
| 7 | Termination Protocol 明确 BOOK_SKILL_INSTALLED 后才能 final 终止 | PASS / FAIL |
| 8 | Qx/10 + 剩余题数规则已写入 private-reading-viva.md | PASS / FAIL |
| 9 | Post-Viva Scoring Protocol 已写入 private-reading-viva.md | PASS / FAIL |
| 10 | Compression Recovery Checkpoint 已写入 skill-state-machine.md | PASS / FAIL |
| 11 | High-risk Truncation Handling 已写入 private-reading-viva.md | PASS / FAIL |
| 12 | Viva Completion Guard 已写入 private-reading-viva.md | PASS / FAIL |
| 13 | Skill Escape Guard 已写入 skill-state-machine.md | PASS / FAIL |
| 14 | State Transition Guard 已写入 skill-state-machine.md | PASS / FAIL |
| 15 | Harness Engineering 文档已创建 | PASS / FAIL |
| 16 | Runtime Trace 已写入 harness-engineering.md | PASS / FAIL |
| 17 | Gate Trace 已写入 harness-engineering.md | PASS / FAIL |
| 18 | Modification Trace 已写入 harness-engineering.md | PASS / FAIL |
| 19 | Eval Harness 已写入 harness-engineering.md | PASS / FAIL |
| 20 | Failure Visibility 已写入 harness-engineering.md | PASS / FAIL |
| 21 | Anti-Goodhart Rule 已写入 harness-engineering.md | PASS / FAIL |
| 22 | Level 3 Candidate 表述已修正（private-reading-viva.md） | PASS / FAIL |
| 23 | 没有创建 book-skill | PASS / FAIL |
| 24 | 没有调用 WeRead API | PASS / FAIL |
| 25 | 没有安装 skill | PASS / FAIL |
