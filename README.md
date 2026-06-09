# Weread → Skill Factory

把微信读书中精读过的书蒸馏成可复用的 book-skill。

---

## 概述

weread-to-skill-factory 是一个本地 Hermes skill factory，用于把微信读书中精读过的书蒸馏成可复用的 book-skill。

**不是平台**：本项目是本地工具，不做平台，不上传用户数据。

**不上传用户数据**：所有数据处理在本地完成，不上传到任何服务器。

**不包含真实 WeRead 数据**：本项目不包含真实 WeRead 数据，只包含虚构示例。

**用户需自行安装 weread skill**：用户需要自行安装 weread skill，并提供自己的 WeRead API key。

**未通过 Reading Gate 不生成 book-skill**：只有通过 Reading Gate 判定的书才能生成 book-skill，确保用户真正精读过。

---

## 功能

### 1. Reading Gate（精读资格判定）

判定用户是否真正精读过某本书。

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

把精读过的书蒸馏成 book-skill。

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

确保 book-skill 质量。

**验证方法**：
- Smoke test：文件结构和内容检查
- Runtime eval：真实 runtime 调用验证
- Forensic audit：独立审计

---

## 安装

### 前置条件

1. **安装 Hermes Agent**
   - 访问 [Hermes Agent 官网](https://hermes-agent.nousresearch.com) 安装
   - 或使用 `pip install hermes-agent`

2. **安装 weread skill**
   - 访问 [agent-weread-skill](https://github.com/lovekeji-ai/agent-weread-skill) 安装
   - 或使用 `hermes skills install agent-weread-skill`

3. **配置 WeRead API key**
   - 访问 [weread.qq.com](https://weread.qq.com) 获取 API key
   - 如果使用 agent-weread-skill，请按 [agent-weread-skill 仓库](https://github.com/lovekeji-ai/agent-weread-skill)的认证流程完成 WeRead 授权
   - 本仓库不包含认证脚本

### 安装步骤

```bash
# 1. 克隆本仓库
git clone https://github.com/panglihaoshuai/hermes-weread-skill-factory.git
cd hermes-weread-skill-factory

# 2. 复制 weread-to-skill-factory 到 ~/.hermes/skills/
cp -R skills/weread-to-skill-factory ~/.hermes/skills/

# 3. 重启 Hermes Agent
hermes gateway restart
```

### 验证安装

```bash
# 检查 skill 是否安装成功
hermes skills list | grep weread-to-skill-factory

# 应该输出：
# weread-to-skill-factory | books | local | local | enabled
```

---

## 使用

### 触发词

- 「蒸馏这本书」
- 「把书变成skill」
- 「distill this book」
- 「book to skill」
- 「从书架里选一本做skill」
- 「做一本XX的skill」
- 「把XX蒸馏成skill」

### 使用流程

#### 1. 选择书目

```
用户：我想把《示例书籍》蒸馏成 skill
```

#### 2. WeRead Evidence Gate

Hermes 会自动：
- 调用 WeRead API 获取你的划线、批注、阅读进度
- 按 20 分制评分（5 维度）
- 判定 Level（0/1/2/3）

#### 3. Viva / Refresher

如果 WeRead 证据不足：
- **Private Reading Viva**：10 题口试，30 分制
- **Refresher Card**：帮你回忆读过的内容

#### 4. Draft Book Skill

通过 Gate 后：
- 生成 Draft Book Skill（标记 DRAFT）
- 你必须批准（WRITE_APPROVED）才能继续

#### 5. Evidence Backfill

如果 Draft 中有 UNKNOWN：
- 自动调用 WeRead API 补齐真实数据
- 不得用 UNKNOWN 数据跑 eval

#### 6. Eval

运行 3 类 eval：
- Trigger / Application / Adversarial evals，具体数量由生成的 book-skill eval 文件决定

#### 7. 用户批准安装

Eval 通过后：
- 你必须再次批准才能安装
- 安装后进入 TERMINATED 状态

---

## 安全边界

### 未通过 Reading Gate 不生成 book-skill

- 不得因为「书架里有」「听过」「热门」「别人推荐」就生成 skill
- 可以总结 ≠ 可以生成 skill
- 只有真正精读过的书才能生成 skill

### 不用热门划线/书评/简介生成 skill

- 必须有个人划线、批注或 Viva 回答
- 只依赖热门划线和公开书评 → Level 0 Reject

### UNKNOWN 不得写 PASS

- API 不可用或未测试时记 UNKNOWN
- UNKNOWN 不得记 0 分
- UNKNOWN 不得直接 PASS

### 不伪造 evidence

- 禁止为了通过测试而隐藏错误
- 禁止编造证据
- 禁止伪造运行过程

### 不自动上传任何数据

- 所有数据处理在本地完成
- 不上传到任何服务器
- 不包含真实 WeRead 数据

---

## 卸载

```bash
# 删除 skill
rm -rf ~/.hermes/skills/weread-to-skill-factory

# 删除 reading cards（可选）
rm -rf ~/.hermes/reading-cards/

# 重启 Hermes Agent
hermes gateway restart
```

---

## 贡献

欢迎贡献代码和文档！

### 贡献方式

1. Fork 本仓库
2. 创建特性分支 (`git checkout -b feature/your-feature`)
3. 提交更改 (`git commit -m 'Add your feature'`)
4. 推送到分支 (`git push origin feature/your-feature`)
5. 创建 Pull Request

### 贡献指南

- 遵循现有代码风格
- 添加测试（如果适用）
- 更新文档（如果适用）
- 确保所有测试通过

---

## 文件结构

```
hermes-weread-skill-factory/
├── README.md
├── LICENSE
├── .gitignore
├── skills/
│   └── weread-to-skill-factory/
│       ├── SKILL.md
│       └── references/
├── examples/
│   └── sample-scope-limited-book-skill/
├── templates/
│   ├── book-skill-template/
│   └── verification-template/
└── docs/
    ├── architecture.md
    ├── harness-engineering.md
    ├── state-machine.md
    └── privacy.md
```

---

## 文档

- [Architecture](docs/architecture.md)：架构设计
- [Harness Engineering](docs/harness-engineering.md)：约束工程
- [State Machine](docs/state-machine.md)：状态机
- [Privacy](docs/privacy.md)：隐私政策

---

## 示例

- [Sample Scope-Limited Book Skill](examples/sample-scope-limited-book-skill/)：虚构示例

---

## 模板

- [Book Skill Template](templates/book-skill-template/)：book-skill 模板
- [Verification Template](templates/verification-template/)：验证模板

---

## 许可证

MIT License - 详见 [LICENSE](LICENSE)

---

## 联系方式

如有问题，请提交 [issue](https://github.com/panglihaoshuai/hermes-weread-skill-factory/issues)。
