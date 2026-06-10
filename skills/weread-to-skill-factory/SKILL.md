---
name: weread-to-skill-factory
version: 1.0
description: >
  把微信读书中精读、划线、批注、认可的书，蒸馏成可复用的 book-skill。
  不是总结书，是提取认知框架、判断规则、反模式和行动协议。
  有 Reading Gate：必须通过精读资格判定才能生成 skill。
  触发词：「蒸馏这本书」「把书变成skill」「distill this book」「book to skill」
  「从书架里选一本做skill」「做一本XX的skill」「把XX蒸馏成skill」。
tags: [book, distillation, skill-creation, weread, knowledge-extraction]
---

# Weread → Skill Factory

把精读过的书蒸馏成可运行的认知操作系统。
不是总结书，不是复述原文，不是写读后感——是提取「遇到问题时如何判断、如何反问、如何行动」。

## ⚠️ Reading Gate（最高优先级）

**没有通过精读资格判定的书，不允许进入 factory 加工为 book-skill。**

核心原则：
- 不要把「书架里有」「听过」「热门」「别人推荐」「简介看起来不错」「热门划线很多」误判成「值得蒸馏为 skill」
- 可以总结 ≠ 可以生成 skill
- 只有真正被用户**精读、理解、认可、批判、迁移**过的书，才允许进入 skill 生成流程

### Gate 判定总流程

```
用户要求蒸馏某本书
    ↓
Reading Gate 检查
    ├─ 路径A: WeRead 证据路径 → 有个人划线/批注/完成度 → 评分
    └─ 路径B: 私下精读自证路径 → 无个人证据 → Private Reading Viva 口试
    ↓
评分判定 → Level 0/1/2/3
    ├─ Level 0: Reject → 不生成任何 skill
    ├─ Level 1: Reading Card → 不安装，不进入 skill 系统
    ├─ Level 2/3 候选 → 用户 WRITE_APPROVED
    │   → 生成 Draft Book Skill（标记 DRAFT）
    │   → 跑 eval（trigger/application/adversarial）
    │   → eval 通过
    │   → 用户再次批准
    │   → 安装正式 book-skill
    └─ Refresher Needed → 输出 Refresher Card 或 Guided Viva 选项
```

### WeRead 证据路径

调用 weread-skills 获取数据。详见 `references/reading-gate-rubric.md`。

**候选发现**：先调 `/user/notebooks` 获取有笔记的书列表（含 reviewCount/noteCount/bookmarkCount），按总笔记数降序选择候选书。不要逐本调 `/book/bookmarklist` 碰运气。

评分采用 20 分制（5维度）：
- 阅读完成度 0-4 / 个人划线密度 0-4 / 个人批注质量 0-5 / 章节覆盖度 0-3 / 现实连接度 0-4

**低分处理规则**：
- 16-20 → Level 3 候选（强通过）
- 12-15 → Level 2 候选（条件通过，要求补充）
- 8-11 → Level 1 Reading Card，但如果用户声称精读过 → 转入 Private Reading Viva
- 0-7 但有个人批注/用户声称精读/外部证据 → 转入 Private Reading Viva
- 0-7 且无任何个人证据且用户不能自证 → Level 0 Reject

**低分不得自动 Reject。** 低分 + 有个人批注/用户自证/外部证据时，必须给用户进入 Viva 的机会。

**硬性失败条件**（任一触发 → 直接 Level 0）：
- 没有个人划线，也没有个人批注，也没有用户自证
- 只依赖热门划线和公开书评
- 用户只是「听说这本书不错」「想以后读」「书架里有」「别人推荐」

**API UNKNOWN 处理**：`/book/getprogress` 和 `/user/notebooks` 当前未验证。API 不可用或未测试时记 UNKNOWN，不得记 0，不得直接 FAIL。详见 `references/weread-data-contract.md`。

### 私下精读自证路径（Private Reading Viva）

如果微信读书缺少个人证据，但用户声称在其他渠道精读过，必须入口试。
三种模式：Strict（10题正式验收）、Guided（逐题追问）、Fixture（模拟测试）。
用户说「忘了/答不上来/题太多」→ Refresher Needed，不得判定伪读。
30 分制，一票否决规则。详见 `references/private-reading-viva.md`。

### 状态机与抗逃逸防护

本 skill 是一个有限状态机（FSM）。一旦进入流程，必须始终处于某个明确状态，不允许处于"无状态"或"自由聊天"状态。

15 个允许状态、状态流转图、终止条件、压缩恢复检查点，详见 `references/skill-state-machine.md`。

**Skill Escape Guard**：进入非 IDLE 状态后，禁止提前结束流程、输出泛泛建议、跳到其他 skill 或自由闲聊。只有达到 TERMINATED 状态才允许退出。

**Compression Recovery**：如果发生 `/compress`、context restore、长对话恢复，必须先输出当前状态和 Required Next Action，再继续执行。

### Harness Engineering（约束工程）

系统必须默认认为「通过测试 ≠ 正确」。禁止为了通过测试而隐藏错误、编造证据、伪造运行过程。

Evidence First Principle、No Fabrication Rule、Runtime Trace、Gate Trace、Modification Trace、Anti-Goodhart Rule 等约束，详见 `references/harness-engineering.md`。

### 产物分级

根据 Gate 结果输出四级产物。详见 `references/output-leveling.md`。

| Level | 名称 | 说明 |
|-------|------|------|
| 0 | Reject | 不生成任何 skill |
| 1 | Reading Card | 概览，不安装 |
| 2 | Draft Book Skill | 草稿，不默认启用 |
| 3 | Installed Book Skill | 正式安装，必须有 evals |

## 蒸馏方法（仅 Level 2/3 适用）

参考女娲五层蒸馏法（`references/nuwa-distillation-pattern.md`）：
表达 DNA → 心智模型 → 决策启发式 → 反模式 → 诚实边界

提取 HOW they think，不是 WHAT they said。

蒸馏输出：一句话总论 + 章节结构地图 + 5-9 个核心模型 + 决策启发式 + 反模式 + 诊断问题 + 行动协议 + 诚实边界。模板见 `references/book-skill-template.md`。

### 增强方法论（不替代女娲五层）

以下方法论用于**增强**蒸馏和 Viva 的深度，不替代女娲五层核心流程。

**核心原则**：
- Reading Gate 仍是唯一入口，不变
- 女娲五层仍是蒸馏骨架，不变
- 以上方法论是"调味料"，让每个环节更有深度

#### 女娲五层 × RIA 集成

| 女娲层级 | RIA 补充 | 解决的问题 |
|---------|---------|-----------|
| 表达DNA | R（原文引用 ≤150字） | 保留作者独特表达 |
| 心智模型 | I（用自己的话重述） | 验证是否真正理解 |
| 决策启发式 | E（1-2-3 可执行步骤） | **补：缺少可执行步骤** |
| 反模式 | A1（书中的失败案例）+ B（边界） | **补：缺少具体案例** |
| 诚实边界 | B（什么时候不该用） | **补：边界不清楚** |

#### A2 触发条件嵌入 skill description

当前 description 可能写得比较泛，用 RIA 的 A2 三层精确化：

1. **场景描述**（3-5条）：用户在什么情况下需要这个
2. **语言信号**：用户会说什么话
3. **边界区分**：和哪些场景不同

**好的 description 示例**：
> 用户在纠结决策、列举理由却理不出头绪时；或在问"怎么做X才能成功"时；
> 不适用于纯信息查询类问题。触发词：纠结、没底、不知道怎么选、先做什么。

**坏的 description 示例**：
> 用户需要思考时。← 太宽泛，会误激活

#### Adler 批判性问题用于蒸馏

在蒸馏"诚实边界"层时，参考 Adler 的批判阶段：
- 作者的时代局限：这本书写于什么时期？哪些前提可能已经不成立？
- 作者的立场盲点：作者的身份/行业/文化背景让他忽略了什么？
- 未被证明的假设：作者把什么当成了不言自明但其实需要论证的东西？
- 反对意见：如果有人要反驳这本书，最强的论点会是什么？

这些批判性问题的答案，直接成为 book-skill 的 `limits.md` 内容。

#### 三重验证用于蒸馏筛选

在女娲五层蒸馏之前，可选执行三重验证（参考 `references/triple-verification.md`）：

- **V1 跨域**：该方法论在书中至少 2 个独立语境有佐证？
- **V2 预测力**：能用它推导出书里没明说的问题答案？
- **V3 独特性**：不是任何聪明人都会说的常识？

通过率预期 25-50%。不通过的降级为 example/引用，不独立成 skill。

详见 `references/ria-tvpp-comparison.md` 和 `references/triple-verification.md`。

## 红线

详见 `references/garbage-prevention.md`。核心：
1. 不得因为出名/书架里有/热门划线多就生成 skill
2. 证据不足时只能生成 reading-card
3. 每个正式 book-skill 必须有 eval + evidence-ledger + 不适用场景

## Pitfalls（已知陷阱）

### Level 3 术语陷阱

**错误**：评分后直接说「Level 3: Installed Book Skill」
**正确**：说「Level 3 Candidate」，并说明「未经 Draft 生成、eval 通过、用户二次批准，不得称为 Installed」

**原因**：Level 3 Candidate 只代表 Gate/Viva 达标，允许进入 Draft 生成流程。Installed 需要完成整个 v1.0 final 流程（Draft → eval → 用户批准安装）。

### v1.0 final 流程陷阱

**错误**：DRAFT_BOOK_SKILL_COMPLETED 后直接 TERMINATED
**正确**：必须继续 EVAL_RUNNING → EVAL_COMPLETED → WAITING_INSTALL_APPROVAL → BOOK_SKILL_INSTALLED → TERMINATED

**原因**：v1.0 final 要求 eval 通过 + 用户二次批准安装。只完成 Draft 不算完成。

### Scope-Limited 声明陷阱

**错误**：声称覆盖全书但实际只有部分章节证据
**正确**：必须声明 `scope-limited`，说明实际覆盖范围和限制

**原因**：WeRead 证据可能只覆盖前几章，Viva 回答可能只涉及部分主题。必须诚实声明范围。

### Post-Viva 泛泛评价陷阱

**错误**：Q10 完成后输出「你阅读很扎实」「建议写读书笔记」
**正确**：必须输出 7 维度评分 + 总分 + 一票否决检查 + 阅读痕迹检查 + Level 判定 + 是否允许 Draft

**原因**：Post-Viva Scoring Protocol 是强制流程，不得跳过或简化。

### /compress 逃逸陷阱

**错误**：/compress 后直接响应新话题，忘记当前状态
**正确**：必须先输出 checkpoint（Current State / Previous State / Required Next Action），再继续执行

**原因**：状态机必须在压缩恢复后立即重建，否则流程丢失。

## API Pitfalls

- **`api_name` 不是 `action`**：WeRead Gateway 的接口名参数是 `api_name`，写错返回 -2010「用户不存在」，容易误判为 key 失效
- **`/review/list/mine` 的参数是 `bookid`（小写 i）**，不是 `bookId`，写错返回 -2003
- **API key 用户绑定可能断开**：key 本身未过期（搜索等公开接口正常），但用户身份绑定失效（/shelf/sync、/user/notebooks、/book/bookmarklist 返回 -2010）。需重新扫码授权（使用 agent-weread-skill 的认证流程，本仓库不包含认证脚本）
- **headless browser 打不开微信读书登录页的 QR 码**：被反爬拦截。用 agent-weread-skill 的 `weread_auth.py --qr` 脚本生成终端 ASCII QR 码 + PNG 文件

## 验证政策

所有检查必须输出 PASS / FAIL / UNKNOWN / SKIPPED。
exit 1 / timeout / blocked / user denied 一律不能写 PASS。
详见 `references/verification-policy.md`。

## 参考文件

| 文件 | 用途 |
|------|------|
| `references/reading-gate-rubric.md` | Reading Gate 详细评分规则（WeRead 20分 + Viva 30分） |
| `references/private-reading-viva.md` | 私下精读口试（10题 + 评分 + 一票否决 + 进度显示 + 截断处理） |
| `references/output-leveling.md` | 产物分级（Level 0-3 详细说明） |
| `references/garbage-prevention.md` | 垃圾场防护规则（10条红线） |
| `references/evidence-ledger-template.md` | 证据账本模板 |
| `references/evidence-rubric.md` | 证据等级标准（A-F） |
| `references/verification-policy.md` | 验证政策（PASS/FAIL/UNKNOWN/SKIPPED + 状态机验证） |
| `references/nuwa-distillation-pattern.md` | 女娲五层蒸馏法参考 |
| `references/weread-data-contract.md` | 微信读书 API 数据契约 |
| `references/book-skill-template.md` | book-skill 文件模板 |
| `references/eval-rubric.md` | 评估评分标准 |
| `references/skill-state-machine.md` | 状态机定义（15状态 + 流转图 + 终止条件 + 压缩恢复 + Install Pipeline + Smoke Test） |
| `references/harness-engineering.md` | 约束工程（Evidence First + No Fabrication + Runtime Trace） |

## 版本门控（v1.0-rc vs v1.0 final）

### v1.0-rc 条件（release candidate）
- Gate 规则定义完整且无冲突
- UNKNOWN 归一化已定义
- 三种 Viva 模式 + Refresher Needed 已定义
- 阅读痕迹覆盖规则已定义
- 已有测试事实记录（Live Gate Dry-run + Fixture Viva）
- 无阻塞性规则冲突

### v1.0 final 条件（在 rc 基础上还需）
- 已生成第一本真实 Draft Book Skill（非 Fixture）
- 已跑完 trigger/application/adversarial eval
- 用户已确认安装正式 book-skill
- `/book/getprogress` 和 `/user/notebooks` API 已 smoke test（或明确标记 UNKNOWN 且不阻塞）

**判定**：v1.0-rc 只靠规则+测试事实；v1.0 final 必须有真实产物。

### v1.0 final 已达成

**日期**：2026-06-09
**第一本正式 book-skill**：mao-zedong-early-thinking-skill（毛泽东早期思想方法论）
**Eval**：19/19 PASS（1 conditional）
**Smoke Test**：32/32 PASS
**Factory Regression**：11/11 PASS
**Factory Version**：1.0

## 流程阶段（v1.0 final 完整流程）

```
IDLE → BOOK_SELECTED → GATE_RUNNING → GATE_COMPLETED
  → GUIDED_VIVA_RUNNING → GUIDED_VIVA_COMPLETED → POST_VIVA_SCORING
  → WAITING_WRITE_APPROVAL → DRAFT_BOOK_SKILL_GENERATING → DRAFT_BOOK_SKILL_COMPLETED
  → EVIDENCE_BACKFILL（补齐 UNKNOWN 证据）→ EVAL_RUNNING → EVAL_COMPLETED
  → WAITING_INSTALL_APPROVAL → BOOK_SKILL_INSTALLED → TERMINATED
```

**Evidence Backfill 阶段**：Draft 生成后，如果 user-highlights.md、user-notes.md、evidence-ledger.md 仍有 UNKNOWN，必须调用 WeRead API 补齐真实数据，再进入 eval。不得用 UNKNOWN 数据跑 eval。

详见 `references/skill-state-machine.md` 和 `references/harness-engineering.md`。

## 常见 Pitfalls

### Pitfall 1：状态计数错误

**问题**：写"允许状态（14 个）"但实际从 IDLE 到 TERMINATED 是 15 个状态。

**原因**：IDLE 从 0 开始编号（0-14），容易少数一个。

**修复**：用 `search_files` 正则 `^\| \d+ \|` 验证实际状态行数。

**教训**：状态机修改后必须做只读验证，不能只数标题。

### Pitfall 2：Level 3 表述不精确

**问题**：写"Level 3 Candidate（允许 Draft + eval + 安装）"，暗示安装已获批准。

**正确表述**：「Level 3 Candidate（允许进入 Draft + eval；安装仍需 eval PASS + 用户二次批准）」

**教训**：Level 3 Candidate 只代表 Gate/Viva 达标，不包含后续批准。每个阶段的权限必须明确分开。

### Pitfall 3：Evidence Backfill 遗漏

**问题**：Draft 生成后直接进入 eval，但 user-highlights.md 和 user-notes.md 仍是 UNKNOWN。

**正确流程**：Draft 完成后检查证据完整性，如有 UNKNOWN 必须先调用 WeRead API 补齐，再进入 eval。

**API 调用顺序**：
1. `/book/bookmarklist` — 获取划线（参数名 `bookId` 驼峰）
2. `/review/list/mine` — 获取批注/想法（参数名 `bookid` 全小写 i）
3. `/book/chapterinfo` — 获取章节目录（参数名 `bookId` 驼峰）

**注意**：`/review/list/mine` 参数名是 `bookid`（小写 i），不是 `bookId`，写错返回 -2003。

### Install Pipeline（安装流程）

Draft → 正式 book-skill 的 5 步安装流程：

1. **RENAME**：`mv draft-skill-dir skill-dir`（去掉 `-draft` 后缀）
2. **UPDATE frontmatter**：name 去掉 `-draft`，version 改 `1.0`，status 改 `INSTALLED`，保留 `scope-limited: true`
3. **REMOVE DRAFT 状态声明**：删除 `**⚠️ DRAFT 状态声明**` 段落，保留 Scope-Limited 声明和不适用场景
4. **UPDATE version info**：版本=1.0，状态=Installed，添加安装日期、Eval Result、Install Permission
5. **UPDATE eval files**：去掉 `⚠️ DRAFT EVAL` 标记

**禁止**：删除 limits.md、evidence-ledger.md、evals/、scope-limited 声明。

### Post-install Smoke Test（安装后冒烟测试）

安装后必须运行 4 类 smoke test，全部 PASS 才能进入 TERMINATED：

1. **File Structure**：验证 14 个必需文件全部存在（SKILL.md + 10 references + 3 evals）
2. **Trigger Smoke Test**：5 个测试（2 个触发 + 2 个拒绝 + 1 个不触发）
3. **Scope Guard**：7 项检查（scope-limited、不覆盖全书、不用于严肃历史定论/政治立场/学术引用、不处理第 7 章以后、证据不足输出 UNKNOWN）
4. **Harness Compliance**：8 项检查（evidence-ledger 存在、limits 存在、eval 可追踪、No Fabrication 生效、UNKNOWN 不写 PASS、State Trace 有记录、Current State 可恢复、Install Permission 有记录）

**Total**: 32 项检查。任何 FAIL 不得进入 TERMINATED。

### Hardening Verification 陷阱

**缺少触发精准度验证（test-prompts.json）**：当前 smoke test + runtime eval + forensic audit 都是验证"skill 内容质量"，缺少"触发时机是否精准"的验证。参考 cangjie-skill 的 test-prompts.json 诱饵测试（should_trigger / should_not_trigger / edge_case 三类），已在 Issue #1 跟踪。集成后可在 eval 阶段增加 trigger precision eval。

**错误**：smoke test 只在 final report 中自证，无独立文件
**正确**：smoke test 必须是独立文件（verification/smoke-test-log.md），与 final report 分离

**错误**：把 `hermes chat -s` 的触发测试写成 "runtime eval"
**正确**：触发测试只验证触发规则（STATIC_RULE_CHECK），完整 runtime eval 需要验证 skill 内部行为

**错误**：`hermes skills inspect` 返回 "No skill named" 就判定 skill 不可用
**正确**：books 类别技能不支持 inspect，但 `hermes chat -s` 可以正常调用

**错误**：runtime 不可用就跳过验证
**正确**：runtime unavailable 时写 RUNTIME_UNAVAILABLE，但 smoke test 和静态 eval 仍需完成，判定为 HARD_VERIFIED_WITH_RUNTIME_UNAVAILABLE

详见 `references/hardening-verification.md`。

### Public Release 脱敏陷阱

**错误**：直接把 ~/.hermes/skills/ 目录推到 public repo
**正确**：创建 sanitized export 目录，替换所有私人数据后才能公开

**错误**：把 `qq.com` 域名当作凭证泄露
**正确**：`weread.qq.com` 是微信读书公开 API 域名，不是私人信息。只有实际的 API key、cookie、token 才是凭证

**错误**：保留真实书名和真实划线原文
**正确**：所有真实书名替换为虚构示例，所有真实划线/批注替换为虚构内容

**错误**：保留 v1.0 final 达成记录（含真实书名和日期）
**正确**：删除包含真实书名的达成记录，或替换为虚构书名

**错误**：示例 book-skill 的 status 写成 INSTALLED
**正确**：示例必须用 `status: SAMPLE`，版本信息中的安装日期写 "N/A（示例）"，Install Permission 写 "N/A（示例）"。避免用户误以为示例是真实安装产物。

**错误**：README 引用其他仓库的脚本（如 `scripts/weread_auth.py`）但不说明来源
**正确**：如果脚本不在本仓库，必须明确写「本仓库不包含X脚本」并链接到实际拥有该脚本的仓库。不要让读者以为当前 repo 提供了该文件。

**错误**：API 文档中同一接口出现多行、状态矛盾（如一行写 "✅ 返回5本"，另一行写 "⚠️ 未测试"）
**正确**：同一接口只保留一行，状态统一为实际验证情况。如果部分环境已验证、部分失败，写明条件和降级策略。

详见 `references/sanitization-for-public-release.md`。

## 降级声明

当 skill-creator 无法完成结构重构/多文件写入时：
1. skill-creator 不支持什么：结构重构、多文件批量写入、旧规则清理
2. 为什么必须降级：文件操作最终必须通过 skill_manage/write_file
3. 替代方案：SKILL.md 用 skill_manage edit，references 用 write_file
4. 会修改哪些文件：列出清单
5. 等待用户确认后才执行

## 参考文件（续）

| 文件 | 用途 |
|------|------|
| `references/post-viva-scoring-protocol.md` | Post-Viva 评分输出模板（7节结构 + 实战经验） |
| `references/ria-tvpp-comparison.md` | RIA-TV++ 方法论对比分析（cangjie-skill 的三重验证、诱饵测试、A2触发精确化，及可借鉴点） |
| `references/enhanced-viva-with-weread-evidence.md` | WeRead 证据增强版 Viva 协议（三层追问：L1证据→L2关联→L3批判应用） |
| `references/weread-troubleshooting.md` | WeRead API 故障诊断（用户绑定失效、key vs binding 区别、re-auth 流程） |
| `references/hardening-verification.md` | Post-install 加固验证（verification 文件结构、runtime eval 方法、forensic audit 检查清单） |
| `references/sanitization-for-public-release.md` | 公开发布前脱敏（grep 检查清单、替换规则、导出目录结构、隐私声明） |

## 修改工作流（必须遵守）

### 审计-修复-审计循环

任何对本 skill 的修改必须遵循三阶段循环：
1. **READ_ONLY_AUDIT**：只读检查，输出问题清单，等待用户批准
2. **WRITE_APPROVED**：用户说「批准修改」后才执行写入
3. **POST-WRITE AUDIT**：写入后立即做只读验证，输出检查表

禁止跳过审计阶段直接写入。

### 流程合规硬性规则

- clarify / approval / confirmation 超时 → **必须停止**，不得自行继续
- 超时后继续执行 → 标记为 **流程合规：FAIL**
- 用户未响应时，最安全选择是停止，不是猜测用户意图
- 降级（从 skill-creator 降到 write_file）必须等用户确认

### 最小补丁模式

当用户说「只修复 X」时：
- 只改 X，不扩大范围
- 不顺手修其他问题
- 不重写整个文件
- 不添加用户未要求的功能

### 设计文档质量要求

当为本 skill 编写设计文档时，**禁止只写说明文档**。设计文档必须包含：

1. **细节模板**：每个增强点的具体模板（字段、格式、示例）
2. **约束**：每个增强点的红线和禁止项（明确列出）
3. **实现案例**：用已安装的 book-skill（如 mao-zedong-early-thinking-skill）作为完整示例
4. **TDD 测试驱动要求**：RED-GREEN-REFACTOR 循环，定义期望行为 → 最小实现 → 验证
5. **Harness Engineering 约束**：Evidence First、No Fabrication、Runtime Trace、Gate Trace、UNKNOWN Discipline

**错误**：写了一个"说明文档"（问题描述 + 方案选择 + 下一步）就算设计文档
**正确**：设计文档必须包含上述 5 项，缺一不可

**原因**：用户对工程质量和可审计性要求很高。粗糙的设计文档会导致实现时缺少约束，产出不可审计的产物。

### CEO Review 检查清单

当使用 gstack-openclaw-ceo-review 审查设计文档时，CEO Review 会检查以下内容：

1. **UNKNOWN 场景定义**：每个增强点必须定义 UNKNOWN 的具体场景（什么情况下输出 UNKNOWN）
2. **回炉流程定义**：每个增强点必须定义回炉流程（触发条件 + 处理方式 + 输出格式）
3. **通过率异常处理**：每个增强点必须定义通过率异常的处理方式（异常定义 + 处理方式 + 输出格式）
4. **指标定义**：每个增强点必须定义具体的指标（指标名称 + 定义 + 计算方式 + 单位）
5. **告警规则**：每个增强点必须定义告警规则（告警名称 + 条件 + 严重程度 + 处理方式）
6. **Runbook**：每个增强点必须定义 Runbook（问题 + 诊断 + 处理 + 验证）

**错误**：设计文档只写了模板、约束、实现案例、TDD、Harness，就认为完整
**正确**：设计文档还必须包含 UNKNOWN 场景、回炉流程、通过率异常处理、指标定义、告警规则、Runbook

**原因**：CEO Review 会检查这些内容，缺失会导致审查不通过。

### Enhanced Viva 设计原则

当设计 Enhanced Viva 时，必须遵守以下原则：

**10 题结构**：
- 证据层（Q1-Q3）：基于用户划线/批注
- 深度层（Q4-Q7）：三重验证 + Adler 批判
- 应用层（Q8-Q10）：RIA A2/E

**自适应规则**：
- 情况 A：划线/批注充足（≥5 条）→ Q1-Q3 用真实划线，Q4-Q10 深度追问
- 情况 B：划线/批注不足（<5 条）→ Q1-Q2 用已有的，Q4-Q10 补充
- 情况 C：划线/批注极少（<2 条）→ 直接从 Q4 开始，用替代问题

**结合用户划线/批注**：
- 优先用用户实际划线/批注提问
- 划线不足时，用三重验证、批判性问题补充
- 目标：验证深度理解，不是验证读过

**错误**：Viva 问题太泛（"这章讲了什么？"）
**正确**：Viva 问题结合用户实际划线/批注，有明确的判断标准和证据要求

**原因**：用户明确要求 Viva 更有深入性和针对性，结合用户微信读书的相关内容和情况。

1. **细节模板**：每个增强点的具体模板（字段、格式、示例）
2. **约束**：每个增强点的红线和禁止项（明确列出）
3. **实现案例**：用已安装的 book-skill（如 mao-zedong-early-thinking-skill）作为完整示例
4. **TDD 测试驱动要求**：RED-GREEN-REFACTOR 循环，定义期望行为 → 最小实现 → 验证
5. **Harness Engineering 约束**：Evidence First、No Fabrication、Runtime Trace、Gate Trace、UNKNOWN Discipline

**错误**：写了一个"说明文档"（问题描述 + 方案选择 + 下一步）就算设计文档
**正确**：设计文档必须包含上述 5 项，缺一不可

**原因**：用户对工程质量和可审计性要求很高。粗糙的设计文档会导致实现时缺少约束，产出不可审计的产物。

### 外部方法论集成原则

当引入外部方法论（如 Adler 分析阅读、RIA 拆书法、RIA-TV++ 三重验证）时：

**核心原则：增强不替代**

- Reading Gate 仍是唯一入口，不变
- 女娲五层仍是蒸馏骨架，不变
- 外部方法论是"调味料"，让每个环节更有深度
- 禁止用外部方法论替换核心流程的任何部分

**错误**：用 RIA-TV++ 的六阶段流水线替换女娲五层蒸馏
**正确**：女娲五层不变，但每一层可以用 RIA 的元素来加深（表达DNA+R、心智模型+I、决策启发式+E、反模式+A1+B、诚实边界+B）

**错误**：用 Adler 分析阅读四步法替换 Reading Gate
**正确**：Reading Gate 不变，但 Viva 问题可以用 Adler 批判性问题来增强深度

**错误**：把三重验证作为蒸馏的必经阶段
**正确**：三重验证是可选的质量筛选层，插在 Reading Gate 之后、女娲五层之前

**原因**：用户明确要求"原流程核心思想不要变"。外部方法论的价值在于增强深度，不在于替换骨架。
