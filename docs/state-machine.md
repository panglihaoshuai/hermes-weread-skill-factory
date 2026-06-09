# State Machine（状态机）

---

## 概述

weread-to-skill-factory 是一个有限状态机（FSM）。一旦进入流程，必须始终处于某个明确状态，不允许处于"无状态"或"自由聊天"状态。

---

## 允许状态（15 个）

| # | 状态 | 说明 |
|---|------|------|
| 0 | IDLE | 空闲状态 |
| 1 | BOOK_SELECTED | 已选书 |
| 2 | GATE_RUNNING | Gate 判定中 |
| 3 | GATE_COMPLETED | Gate 判定完成 |
| 4 | GUIDED_VIVA_RUNNING | Viva 口试中 |
| 5 | GUIDED_VIVA_COMPLETED | Viva 口试完成 |
| 6 | POST_VIVA_SCORING | Viva 后评分 |
| 7 | WAITING_WRITE_APPROVAL | 等待写入批准 |
| 8 | DRAFT_BOOK_SKILL_GENERATING | Draft 生成中 |
| 9 | DRAFT_BOOK_SKILL_COMPLETED | Draft 生成完成 |
| 10 | EVIDENCE_BACKFILL | 证据补齐 |
| 11 | EVAL_RUNNING | Eval 运行中 |
| 12 | EVAL_COMPLETED | Eval 完成 |
| 13 | WAITING_INSTALL_APPROVAL | 等待安装批准 |
| 14 | BOOK_SKILL_INSTALLED | Book Skill 已安装 |
| 15 | TERMINATED | 终止 |

---

## 状态流转图

```
IDLE
  ↓
BOOK_SELECTED
  ↓
GATE_RUNNING
  ↓
GATE_COMPLETED
  ├─ Level 0: Reject → TERMINATED
  ├─ Level 1: Reading Card → TERMINATED
  └─ Level 2/3: → GUIDED_VIVA_RUNNING
                    ↓
                  GUIDED_VIVA_COMPLETED
                    ↓
                  POST_VIVA_SCORING
                    ↓
                  WAITING_WRITE_APPROVAL
                    ↓
                  DRAFT_BOOK_SKILL_GENERATING
                    ↓
                  DRAFT_BOOK_SKILL_COMPLETED
                    ↓
                  EVIDENCE_BACKFILL
                    ↓
                  EVAL_RUNNING
                    ↓
                  EVAL_COMPLETED
                    ↓
                  WAITING_INSTALL_APPROVAL
                    ↓
                  BOOK_SKILL_INSTALLED
                    ↓
                  TERMINATED
```

---

## 终止条件

### 正常终止

- Level 0 Reject → TERMINATED
- Level 1 Reading Card → TERMINATED
- BOOK_SKILL_INSTALLED → TERMINATED

### 异常终止

- 用户明确要求停止
- 系统错误
- 超时

---

## 压缩恢复检查点

如果发生 `/compress`、context restore、长对话恢复，必须先输出：

```
Current State: [当前状态]
Previous State: [上一个状态]
Required Next Action: [下一步动作]
```

再继续执行。

---

## Skill Escape Guard

进入非 IDLE 状态后，禁止：
- 提前结束流程
- 输出泛泛建议
- 跳到其他 skill
- 自由闲聊

只有达到 TERMINATED 状态才允许退出。

---

## Compression Recovery

如果发生 `/compress`、context restore、长对话恢复，必须先输出当前状态和 Required Next Action，再继续执行。

---

## Install Pipeline

Draft → 正式 book-skill 的 5 步安装流程：

1. **RENAME**：`mv draft-skill-dir skill-dir`（去掉 `-draft` 后缀）
2. **UPDATE frontmatter**：name 去掉 `-draft`，version 改 `1.0`，status 改 `INSTALLED`，保留 `scope-limited: true`
3. **REMOVE DRAFT 状态声明**：删除 `**⚠️ DRAFT 状态声明**` 段落，保留 Scope-Limited 声明和不适用场景
4. **UPDATE version info**：版本=1.0，状态=Installed，添加安装日期、Eval Result、Install Permission
5. **UPDATE eval files**：去掉 `⚠️ DRAFT EVAL` 标记

**禁止**：删除 limits.md、evidence-ledger.md、evals/、scope-limited 声明。

---

## Post-install Smoke Test

安装后必须运行 4 类 smoke test，全部 PASS 才能进入 TERMINATED：

1. **File Structure**：验证 14 个必需文件全部存在
2. **Trigger Smoke Test**：5 个测试
3. **Scope Guard**：7 项检查
4. **Harness Compliance**：8 项检查

**Total**: 32 项检查。任何 FAIL 不得进入 TERMINATED。
